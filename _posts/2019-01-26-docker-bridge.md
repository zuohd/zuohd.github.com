---
layout: post
title:  Docker之Bridge
date:   2019-01-26 13:35:48 +0800
categories: technology
tags: [DOCKER]
---
1.查看本地docker网络
```shell
sudo docker network ls
```
>输出结果
```output
NETWORK ID          NAME                DRIVER              SCOPE
01e1710ea79f        bridge              bridge              local
44b555f0100a        host                host                local
9bc93e885e42        none                null                local
```

2.查看bridge网络连接情况
``` shell
docker network inspect 01e1710ea79f
```
输出结果中的container部分有
>"Containers": {
    "a463215111f59f15a9046580632e7ab63fb819ef6cc89e33c64b3d31630e5d7f": {
        "Name": "test1",
        "EndpointID": "2f0d203cef6656a4a343c0fa5be01a8ff8d5f1b5bae82f129016e89f39c7f02a",
        "MacAddress": "02:42:ac:11:00:02",
        "IPv4Address": "172.17.0.2/16",
        "IPv6Address": ""
    }
}

代表test1连接的是此桥接网络

3.查看桥接网络以及接口-该命令需要安装bridge-utils

```shell
 brctl show
```
>输出结果
```output
bridge name     bridge id               STP enabled     interfaces
docker0         8000.0242d91853ea       no              veth50a2bda
```
由此可见容器与主机之间的通讯是基于虚拟ethnet原理来进行的
