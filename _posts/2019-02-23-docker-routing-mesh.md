---
layout: post
title:  Docker routing mesh 机制
date:   2019-02-23 16:08:48 +0800
categories: technology
tags: [Docker,Docker swarm]
---

前面我们讲到当我们通过新建的 overlay driver 的网络后，docker swarm运用了VIP技术，那我们如何验证呢？
# 体现为Internal
1.进入 Manager 主机新建 whoami 服务：

``` shell
docker service create  --name whoami -p 8000:8000  --network demo -d jwilder/whoami
```
2.建立另一服务命名为 client:

``` shell
 docker service create --name client -d --network demo busybox sh -c "while true;do sleep 3600;done"
```
3.运行`docker service ps whoami`和`docker service ps client`查看服务所在主机：

``` shell
[vagrant@node-1 ~]$ docker service ps whoami
ID                  NAME                IMAGE                   NODE                DESIRED STATE       CURRENT STATE
         ERROR               PORTS
ks374hlespq4        whoami.1            jwilder/whoami:latest   node-3              Running             Running 17 secon
ds ago

[vagrant@node-1 ~]$  docker service ps client
ID                  NAME                IMAGE               NODE                DESIRED STATE       CURRENT STATE
         ERROR               PORTS
d36f6825mxnc        client.1            busybox:latest      node-1              Running             Running about a minu
te ago
```
我们发现 client 服务在 node1 机器上，而 whoami 在 node3 机器上，我们通过 `docker ps` 查看client 容器ID，然后进入client，执行 `ping whoami`得到IP地址为 `10.0.0.8`。我们在node3机器上执行`docker ps`查看 whomai 容器ID，执行`ip a`并没有找到 `10.0.0.8`,由此可知 `10.0.0.8` 是虚拟IP，执行`nslookup tasks.whomai` 才可以得到容器的 IP地址，还可以通过`docker service scale whoami=3`去扩展 whoami 服务，再次执行`nslookup tasks.whoami` 会发现3个IP地址，但都会映射到 VIP `10.0.0.8`。
`docker exec  -it e387 sh`进入client容器,将以下语句多执行几次会得到不同的结果：

``` shell
wget whoami:8000
more index.html #I'm 6f05266a5ecc

rm -rf index.html
wget whoami:8000
more index.html #I'm e7f3923493fe
```
此处 VIP 还担负了负载均衡的职能。
# 体现为Ingress
1.将 whoami 服务 scale 减少为2：

``` shell
docker service scale whoami=2
```
通过`docker service ps whoami`查看到服务部署在了node2和node3上，我们进入node2主机多次执行`curl 127.0.0.1:8000`发现每次结果不同，然后进入node1机器，执行`curl 127.0.0.1:8000`同样成功取得结果。

2.node1上执行`sudo iptables -nL -t nat`,部分结果如下:

``` shell
Chain DOCKER-INGRESS (2 references)
target     prot opt source               destination
DNAT       tcp  --  0.0.0.0/0            0.0.0.0/0            tcp dpt:8000 to:172.18.0.2:8000
DNAT       tcp  --  0.0.0.0/0            0.0.0.0/0            tcp dpt:8080 to:172.18.0.2:8080
RETURN     all  --  0.0.0.0/0            0.0.0.0/0
```
3.查看虚拟网卡命令，结果显示有一个`docker_gwbridge`：

```shell
brctl show
```
4.查看连接到`docker_gwbridge`的容器：

```shell
docker network inspect docker_gwbridge
```
结果显示有两个容器连接到该网络
``` output
"Containers": {
    "030aed15efe6336be61e157892e65ae220e2147b02eac6b0ef7dda5b22771c95": {
        "Name": "gateway_7df29b7cce21",
        "EndpointID": "729f561312f8994995027a8e6aa5d6f48d341e285b48a9de7a61344e195fe806",
        "MacAddress": "02:42:ac:12:00:03",
        "IPv4Address": "172.18.0.3/16",
        "IPv6Address": ""
    },
    "e3876aa4eb81a0e05c3c7aeaccb5e1139e9ef3e01213ea7e347eee7255d5330a": {
        "Name": "gateway_96da5f57a0ef",
        "EndpointID": "8cfb89c3ddf3615b300ccd1abd0e1547180d3f1a03023b610658dc89e5216453",
        "MacAddress": "02:42:ac:12:00:04",
        "IPv4Address": "172.18.0.4/16",
        "IPv6Address": ""
    },
    "ingress-sbox": {
        "Name": "gateway_ingress-sbox",
        "EndpointID": "ff58ee04454bfab7065e6f76bb343cb0f3af86bbb51901267e7f80801536679c",
        "MacAddress": "02:42:ac:12:00:02",
        "IPv4Address": "172.18.0.2/16",
        "IPv6Address": ""
    }
}

```
执行一下`sudo ls /var/run/docker/netns`查看网络命名空间：
```output
1-sdvu0synsc  1-wo94n9z3rj  7df29b7cce21  96da5f57a0ef  ingress_sbox  lb_sdvu0syns
```
5.进入网络命名空间`ingress_sbox`:

```shell
sudo nsenter --net=/var/run/docker/netns/ingress_sbox
```
执行 `iptables -nL -t mangle` 查看iptables的mangle表：

``` output
Chain PREROUTING (policy ACCEPT)
target     prot opt source               destination
MARK       tcp  --  0.0.0.0/0            0.0.0.0/0            tcp dpt:8080 MARK set 0x102
MARK       tcp  --  0.0.0.0/0            0.0.0.0/0            tcp dpt:8000 MARK set 0x104
```
可以看到8000端口做了标记 MARK

6.通过 LVS 管理工具（yum install -y ipvsadm）查看内核中的虚拟服务：

``` shell
[root@node-1 vagrant]# ipvsadm -l
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
FWM  258 rr
  -> 10.255.0.6:0                 Masq    1      0          0
FWM  260 rr
  -> 10.255.0.8:0                 Masq    1      0          0
  -> 10.255.0.9:0                 Masq    1      0          0
```
在部署 whoami 的 node2 机器我们执行`docker exec e7f3 ip a`，可以看到 ip 是 `10.255.0.9`,
在部署 whoami 的 node3 机器我们执行`docker exec 61b2 ip a`，可以看到 ip 是 `10.255.0.8`