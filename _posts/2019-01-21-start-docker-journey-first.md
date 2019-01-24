---
layout: post
title:  "启动docker实践之旅"
date:   2019-01-21 08:35:48 +0800
categories: DOCKER
tags: [technology]
author: Soderberg
---
我这里使用了[vagrant][vagrant-link]和[Virtual Box][VirtualBox-link]作为实验环境,其使用方法参照了博客文章[路径（七）：用 Vagrant 管理虚拟机][use-vagrant],另外安装完毕后，需要启用计算机Bios的`intel virtual technology`,并在windows控制面板中卸载hyper-v,box文件资源可以网上下载，这样安装会比较快。

### 一、修改生成的vagrantfile，配置其vmbox名称和虚拟机启动后执行的脚本(安装docker)

``` vagrantfile

config.vm.box = "centos-7"  #=>名字需要与之前定义的vagrant box名称一致

--------------------------黄金分割线-------------------------

 config.vm.provision "shell", inline: <<-SHELL
     sudo yum remove docker docker-client docker-client-latest docker-common docker-latest docker-latest-logrotate docker-logrotate docker-engine
	 sudo yum install -y yum-utils device-mapper-persistent-data lvm2
	 sudo yum-config-manager -y --add-repo https://download.docker.com/linux/centos/docker-ce.repo
	 sudo yum install -y docker-ce
	 sudo systemctl start docker
   SHELL
end
```
1.将vagrant用户添加到docker用户组

``` shell

sudo groupadd docker
sudo gpasswd -a vagrant docker

```

2.退出登录，重启docker服务

``` shell

exit
sudo service docker restart

```
3.查看docker镜像

``` shell

docker image ls

```
4.查看docker容器
``` shell

docker container ls

```
5.参考[Docker pull 出现的TLS handshake timeout][docker-pull-tls-handshake-timeout]执行以下命令加速docker镜像下载

``` shell
echo "DOCKER_OPTS=\"\$DOCKER_OPTS --registry-mirror=http://f2d6cb40.m.daocloud.io\"" | sudo tee -a /etc/default/d
ocker
```

6.获取docker镜像，如nginx
``` shell

docker pull nginx

```

当然我们也可以获取指定版本或tag的镜像

``` shell

docker pull ubuntu:14.04 

```
### 二、 构建一个简单的docker image
1.参照[创建一个c文件][C-compile-process],命名为hello.c并编译


2.创建dockfile
``` dockfile
FROM scratch
ADD hello /
CMD ["/hello"]
```
3.编译
``` shell
docker build -t soderberg/hello-world .
```
4.运行container
``` shell
docker run soderberg/hello-world
```
5.查看docker镜像的层
```shell
docker history 928b474fce0f #=>928b474fce0f 对应的是IMAGE ID
```
>查看执行结果

``` output
[vagrant@localhost hello-world]$ docker history 928b474fce0f
IMAGE               CREATED              CREATED BY                                      SIZE                COMMENT
928b474fce0f        About a minute ago   /bin/sh -c #(nop)  CMD ["/hello"]               0B
63f27a030eb2        About a minute ago   /bin/sh -c #(nop) ADD file:589e7a47dcdc1f1bd…   861kB
```
### 三、 docker container 的操作
1.守护方式启动container

```shell
docker run -d zuohd/flask-demo
```
2.进入容器执行命令

``` shell
docker exec -it ebfa510c1e0b /bin/bash
```
3.列出运行的容器IP地址
``` shell
docker exec -it ebfa510c1e0b ip a
 
```
>查看执行结果

``` output
[root@localhost flask-hello-world]# docker exec -it ebfa510c1e0b ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
38: eth0@if39: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:ac:11:00:03 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.17.0.3/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever
```
4.停止容器

``` shell
docker stop ebfa510c1e0b
```
5.启动容器
``` shell
docker start ebfa510c1e0b
```
6.移除所有容器实例
``` shell
docker rm $(docker ps -aq)
```
7.给docker容器实例命名
``` shell
docker run -d --name=appserver zuohd/flask-demo
```
8.查看容器详细信息
``` shell
docker inspect a51b112c4f60
``` 
9.查看容器日志信息
``` shell
docker logs -f a51b112c4f60
``` 

[vagrant-link]:https://www.vagrantup.com/
[VirtualBox-link]:https://www.virtualbox.org/
[use-vagrant]:https://ninghao.net/blog/2077
[docker-pull-tls-handshake-timeout]:https://blog.kelu.org/tech/2017/02/08/docker-pull-tls-handshake-timeout.html
[C-compile-process]:https://zuohd.github.io/programming/2017/07/11/C-compile-process.html

