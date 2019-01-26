---
layout: post
title:  Linux 网络命名空间之相互通讯
date:   2019-01-26 08:35:48 +0800
categories: technology
tags: [Linux shell]

---
Docker 容器之间之所以能通讯，也是利用了网络命名空间的原理，我们可以创建两个容器，然后使用命令`docker exec 6b51903a07f5 ip a`查看容器的IP地址。
### 一、测试容器之间的通讯
1.创建容器 `test1`/`test2`
```shell
docker run -d --name=test1  busybox /bin/sh -c "while true;do sleep 3600;done"
docker run -d --name=test2  busybox /bin/sh -c "while true;do sleep 3600;done"
```
2.`docker ps`列出运行的容器
  >输出结果
``` output
[vagrant@soderberg-dev ~]$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS
     NAMES
6b51903a07f5        busybox             "/bin/sh -c 'while t…"   6 seconds ago       Up 5 seconds
     test2
a463215111f5        busybox             "/bin/sh -c 'while t…"   5 minutes ago       Up 5 minutes
     test1
```

3.测试网络是否联通
``` shell
docker exec -it a463215111f5 /bin/sh
/ # ip a
```
 >输出结果
``` output 
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
11: eth0@if12: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue
    link/ether 02:42:ac:11:00:02 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.2/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever
```

4.执行`ping 172.17.0.3`查看结果

### 二、网络命名空间之通讯

1.查看以及存在的网络命名空间
```shell
 ip netns list
```
2.建立两个网络命名空间
```shell
sudo ip netns add test1
sudo ip netns add test2
```
3.进入网络命名空间`test1`查看IP地址
```shell
sudo ip netns exec test1 ip a
```
4.查看IP连接
```shell
ip link
```
5.启动test1的l0网络状态

```shell
sudo ip netns exec test1 ip link set dev lo up
```

6.添加veth对到当前主机,且名为`veth-test1`和`veth-test2`
```shell
sudo ip link add veth-test1 type veth  peer name veth-test2
```
7.将veth-test1设备添加到网络命名空间test1，veth-test2添加到网络命名空间test2
```shell
sudo ip link set veth-test1 netns test1
sudo ip link set veth-test2 netns test2
```
8.设置IP地址
```shell
sudo  ip netns exec test1 ip addr add 192.168.1.1/24 dev veth-test1
sudo  ip netns exec test2 ip addr add 192.168.1.2/24 dev veth-test2
```
9.启动设备`veth-test1`和`veth-test2`
```shell
sudo ip netns exec test1 ip link set dev veth-test1 up
sudo ip netns exec test2 ip link set dev veth-test2 up
```
10.测试设备之间是否联通

```shell
sudo ip netns exec test1 ping 192.168.1.2
sudo ip netns exec test2 ping 192.168.1.1
```

