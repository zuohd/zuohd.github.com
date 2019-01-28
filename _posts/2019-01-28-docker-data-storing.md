---
layout: post
title:  Docker数据存储
date:   2019-01-28 14:35:48 +0800
categories: technology
tags: [Docker,Docker volume]
---
### 一、Docker之卷标volume
Volume我们可以在dockerfile中指定，保证容器删除后数据不丢失。

1.运行mysql容器
``` shell
docker run -d --name mysql1 -e MYSQL_ALLOW_EMPTY_PASSWORD=Yes mysql
```
2.查看docker本地卷标
``` shell
docker volume ls
```
``` output
DRIVER              VOLUME NAME
local               bcece002b3eba18c8844f19920a621b0b0aab549f302b29a4451b70520a8eeb8
```
3.指定volumn位置
```shell
docker run -d -v mysql:/var/lib/mysql --name mysql2 -e MYSQL_ALLOW_EMPTY_PASSWORD=Yes mysql
```
4.进入mysql
```shell
docker exec -it mysql2 /bin/bash
mysql -u root
```
5.创建数据库
```mysql
 create database docker;
```
```output
mysql> create database docker;
Query OK, 1 row affected (0.09 sec)

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| docker             |
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.00 sec)

```
6.删除容器mysql2，新建容器mysql3并制定volume为mysql2的
```shell
 docker run -d -v mysql:/var/lib/mysql --name mysql3 -e MYSQL_ALLOW_EMPTY_PASSWORD=Yes mysql
```
进入mysql3,查询数据库发现数据没有丢失:
```shell
docker exec -it mysql3 /bin/bash
mysql -u root
mysql> show databases;
```
### 二、Docker之Bind Mouting

```shell
docker run -d -v $(pwd):/usr/share/nginx/html -p 80:80 --name nginx-web zuohd/ngin
x-hello
```
主机当前目录将和docker容器的`/usr/share/nginx/html`数据同步,利用此机制我们可以将我们的源码目录与服务器部署目录同步调试，减少了开发人员的工作量。