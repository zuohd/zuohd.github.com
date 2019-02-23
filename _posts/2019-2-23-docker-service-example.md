---
layout: post
title:  集群部署wordpress博客
date:   2019-02-23 14:08:48 +0800
categories: technology
tags: [Docker,Docker swarm]
---
前面我们提到docker swarm环境我们要使用`docker service`命令部署docker容器，今天我还以wordpress为例说明。

1.`vagrant ssh swarm-manager`进入node1创建network使得容器间可以通讯：

```shell
docker network create -d overlay demo
```

2.创建mysql服务：

``` shell
[vagrant@node-1 ~]$ docker service create --name mysql --env MYSQL_ROOT_PASSWORD=root --env MYSQL_DATABASE=wordpress --
network demo --mount type=volume,source=mysql-data,destination=/var/lib/mysql mysql:5.7
```
2.创建wordpress服务(-p参数格式为 `hostport:containerport` ，当然也可以用-P让docker随机映射)：

``` shell
[vagrant@node-1 ~]$ docker service create --name wordpress -p 8080:80 --env WORDPRESS_DB_PASSWORD=root --env WORDPRESS_
DB_HOST=mysql --network demo wordpress
```
分别执行`docker service ps mysql`和`docker service ps wordpress`可以看到mysql在机器node1上，wordpress在机器node2上，`vagrant ssh swarm-worker1`进入node2机器，`ip -a`查看机器ip为`192.168.205.11`,通过浏览器访问`192.168.205.11：8080`出现wordpress安装界面，安装完成后程序可以正常访问说明数据库连接也是成功的。

>同时会注意到通过 node1 和 node3 的机器 IP 加 8080 端口依然可以访问 wordpress 程序，为什么呢？DNS发现服务，当建立了overlay网络，docker swarm引擎会建立虚拟IP即VIP。
