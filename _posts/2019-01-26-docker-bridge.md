---
layout: post
title:  Docker网络连接之bridge
date:   2019-01-26 13:35:48 +0800
categories: technology
tags: [Docker,Bridge]
---

*内容概览*

* Do not remove this line (it will not be displayed)
{:toc}

# 一、Docker Bridge连接查询

1.查看本地docker网络

```shell
sudo docker network ls
```

输出结果如下：

| NETWORK ID   | NAME  | DRIVER | SCOPE |
| ------------ | :----: | :----: | ----: |
| 01e1710ea79f | bridge | bridge | local |
| 44b555f0100a | host  | host  | local |
| 9bc93e885e42 | none  | null  | local |

2.查看bridge网络连接情况

``` shell
docker network inspect 01e1710ea79f
```

输出结果中的container部分有

```json
"Containers": {
    "a463215111f59f15a9046580632e7ab63fb819ef6cc89e33c64b3d31630e5d7f": {
        "Name": "test1",
        "EndpointID": "2f0d203cef6656a4a343c0fa5be01a8ff8d5f1b5bae82f129016e89f39c7f02a",
        "MacAddress": "02:42:ac:11:00:02",
        "IPv4Address": "172.17.0.2/16",
        "IPv6Address": ""
    }
}
```

代表test1连接的是此桥接网络

3.查看桥接网络以及接口-该命令需要安装bridge-utils

```shell
 brctl show
```

输出结果如下：

| bridge name | bridge id         | STP enabled  | interfaces  |
| ---------   | :---------------: | :----------: | ---------:  |
| docker0     | 8000.0242d91853ea | no           | veth50a2bda |

由此可见容器与主机之间的通讯是基于虚拟ethnet原理来进行的

# 二、容器之间的link

如果我们要建立到某容器的连接，容器IP我们是不知道的，当我们可以实现指定要连接的容器名称，利用docker link命令实现。如有一台mysql的容器，名称暂定为`mysqldb`。

1.启动app容器，名为`appServer1`

```shell
docker run -d --name appServer1 --link mysqldb busybox /bin/sh -c "while true;do sleep 3600;done"
```

2.测试数据库容器连接

```shell
docker exec appServer1 ping mysqldb
```

结果显示成功，作用上类似建立了dns解析，注意此处的link是单方向的，反之我们要在mysqldb容器中连接appServer1则不可以。

# 三、设置容器连接网络

1.新建一个bridge网络

```shell
docker network create my-bridge
```

2.`docker network ls`查看结果

| NETWORK ID    |      NAME      |    DRIVER   | SCOPE |
| --|:--:|:--:|--:|
|01e1710ea79f     |   bridge     |    bridge   | local |
|44b555f0100a     |  host        |    host     | local |
|4e5b148454ef     |  my-bridge   |    bridge   | local |
|9bc93e885e42     |  none        |    null     | local |

3.让容器以指定的Network运行

``` shell
docker run -d --name test3 --network my-bridge busybox /bin/sh -c "while true;do sleep 3600;
done"
```

4.`brctl show`查看桥接网络

|bridge name   |  bridge id         |  STP enabled   |  interfaces|
|--|:--:|:--:|--:|
|br-4e5b148454ef | 8000.02429f7f61fb | no     |         vethe1cf9d9|
|docker0        | 8000.0242d91853ea  |   no   | veth0f20a56 <br /> veth50a2bda|

5.查看指定网络连接的容器

``` shell
docker inspect network 4e5b148454ef
```

输出结果的container部分内容:
``` json
 "Containers": {
    "975fdb5637084574b29ca8a577489937ba28df2f315ade8b4b7552a7d07c80c3": {
        "Name": "test3",
        "EndpointID": "e684184f2b243b67d08d8063c502e73105f444a635dcf1a309a9c1f2bc92ceaf",
        "MacAddress": "02:42:ac:12:00:02",
        "IPv4Address": "172.18.0.2/16",
        "IPv6Address": ""
    }
}
```

由此我们得知容器test3连接在该网络上，其IP地址为172.18.0.2

6.将已有的容器test2连接到指定网络

``` shell
docker network connect  my-bridge test2
```

再次运行`docker inspect network 4e5b148454ef`查看输出结果中的container部分内容：

```json
"Containers": {
    "4c6ed6849d4b884b15ddbafb8e8e2b620f53844689132d256dc07bf3bd68cf73": {
        "Name": "test2",
        "EndpointID": "dcec7a7ffa60b16ad1c13d687217b747b569de63c7f24a6f9132edd26cdcd9d2",
        "MacAddress": "02:42:ac:12:00:03",
        "IPv4Address": "172.18.0.3/16",
        "IPv6Address": ""
    },
    "975fdb5637084574b29ca8a577489937ba28df2f315ade8b4b7552a7d07c80c3": {
        "Name": "test3",
        "EndpointID": "e684184f2b243b67d08d8063c502e73105f444a635dcf1a309a9c1f2bc92ceaf",
        "MacAddress": "02:42:ac:12:00:02",
        "IPv4Address": "172.18.0.2/16",
        "IPv6Address": ""
    }
}
```

我们看到test3和test1都连到了同一网络，同时我们查看docker0网络的容器连接情况：

``` shell
docker network inspect 01e1710ea79f
```

输出结果中的container部分内容显示：

``` json
"Containers": {
    "4c6ed6849d4b884b15ddbafb8e8e2b620f53844689132d256dc07bf3bd68cf73": {
        "Name": "test2",
        "EndpointID": "f7a42fa04dc764aa3401b290712569871450b0eabc5561ebb64042796e7e293e",
        "MacAddress": "02:42:ac:11:00:03",
        "IPv4Address": "172.17.0.3/16",
        "IPv6Address": ""
    },
    "a463215111f59f15a9046580632e7ab63fb819ef6cc89e33c64b3d31630e5d7f": {
        "Name": "test1",
        "EndpointID": "2f0d203cef6656a4a343c0fa5be01a8ff8d5f1b5bae82f129016e89f39c7f02a",
        "MacAddress": "02:42:ac:11:00:02",
        "IPv4Address": "172.17.0.2/16",
        "IPv6Address": ""
    }
}
```

看来test2同时连入了两个网络docker0和my-bridge。不过我们要想成功运行`ping test1`,还需要将test1容器也连入我们的自定义网络才可以：

 ``` shell
 docker network connect my-bridge test1
 ```
