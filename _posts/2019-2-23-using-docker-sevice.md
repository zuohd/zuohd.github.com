---
layout: post
title:  Docker Service命令使用
date:   2019-02-23 09:08:48 +0800
categories: technology
tags: [Docker,Docker swarm]
---

在集群环境下我们要使用`docker service`命令创建docker容器了

1.创建name为demo的docker service：

``` shell
docker service create --name demo busybox sh -c "while true;do sleep 3600;done"
```

2.查看docker service,`REPLICAS`代表可以水平扩展：

```shell
[vagrant@node-1 ~]$ docker service ls
ID                  NAME                MODE                REPLICAS            IMAGE               PORTS
dg78z3vvt13p        demo                replicated          1/1                 busybox:latest

```

3.查看docker service demo的运行情况：

``` shell
docker service ps demo
```

从输出结果我们可以看到其运行在manager即node1节点上，我们再通过`docker ps`命令看一下，

``` shell
[vagrant@node-1 ~]$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS
     NAMES
d8d48244dc67        busybox:latest      "sh -c 'while true;d…"   5 minutes ago       Up 5 minutes
     demo.1.kdnwfemo37jbntde8vtmnmt6x
```

4.我们可以通过scale命令让demo服务水平扩展为5个：

```shell
docker service scale demo=5
```

5.`docker service ps`查看`REPLICAS`为`5/5`,分母代表总个数，分子代表状态ready的数量:

``` shell
[vagrant@node-1 ~]$ docker service ls
ID                  NAME                MODE                REPLICAS            IMAGE               PORTS
dg78z3vvt13p        demo                replicated          5/5                 busybox:latest
```

6.运行`docker service ps demo`查看`NODE`字段我们可以看到服务分配情况：

``` shell
[vagrant@node-1 ~]$ docker service ps demo
ID                  NAME                IMAGE               NODE                DESIRED STATE       CURRENT STATE
     ERROR               PORTS
kdnwfemo37jb        demo.1              busybox:latest      node-1              Running             Running 16 minutes a
go
cvoghm5kcgrh        demo.2              busybox:latest      node-3              Running             Running 6 minutes ag
o
wh3w8s9l8bl3        demo.3              busybox:latest      node-1              Running             Running 6 minutes ag
o
mm946n46m153        demo.4              busybox:latest      node-2              Running             Running 6 minutes ag
o
6y1k1s2rt1lx        demo.5              busybox:latest      node-3              Running             Running 6 minutes ag
o
```

此时我们可以`vagrant ssh`进入node2和node2机器分别运行`docker ps`可以看到相应个数的demo服务已经启动。

7.试着删除node3节点的一个容器：

``` shell
docker rm -f 36b627aa6cc7
```

运行`docker service ls`会看到`REPLICAS`变为`4/5`,说明有一个容器挂了，几秒后再次运行`docker service ls`会看到`REPLICAS`恢复为`5/5`，证明scale有确保集群内有效服务数运行的功能。

8.在manager节点删除demo服务，其集群内的相关服务也将被删除：

```shell
[vagrant@node-1 ~]$ docker service rm demo
```