---
layout: post
title:  Docker网络连接之None和Host
date:   2019-01-28 12:35:48 +0800
categories: technology
tags: [Docker,Docker None,Docker host]
---
*内容概览*

* Do not remove this line (it will not be displayed)
{:toc}

# 一、指定容器以None网络连接方式运行

``` shell
 docker run -d --name test --network none busybox /bin/sh -c "while true;do sleep 3600;done"
```

查看其IP地址发现没有配置网络地址

```shell
docker exec  test ip a
```

***
输出结果如下：

1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever


# 二、指定容器以Host网络连接方式运行

``` shell
 docker run -d --name test --network none busybox /bin/sh -c "while true;do sleep 3600;done"
```

查看其IP地址发现其与本地主机网络配置相同

```shell
docker exec  test ip a
```

另行补充一个小知识，我们还可以指定容器运行的环境变量

``` shell
docker run -d --name test -e hostowner=soderberg busybox /bin/sh -c "while true;do sleep 360
0;done"
```

利用env命令查看环境变量

``` output
/ # env
HOSTNAME=814e2d4f46f4
SHLVL=1
HOME=/root
hostowner=soderberg
TERM=xterm
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
PWD=/
```
