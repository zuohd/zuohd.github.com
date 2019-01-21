---
layout: post
title:  "启动docker实践之旅"
date:   2019-01-21 08:35:48 +0800
categories: DOCKER
tags: [technology]
---
我这里使用了[vagrant][vagrant-link]和[Virtual Box][VirtualBox-link]作为实验环境,其使用方法参照了博客文章[路径（七）：用 Vagrant 管理虚拟机][use-vagrant],另外安装完毕后，需要启用计算机Bios的`intel virtual technology`,并在windows控制面板中卸载hyper-v,box文件资源可以网上下载，这样安装会比较快。
[vagrant-link]:https://www.vagrantup.com/
[VirtualBox-link]:https://www.virtualbox.org/
[use-vagrant]:https://ninghao.net/blog/2077

## 修改生成的vagrantfile，配置其vmbox名称和虚拟机启动后执行的脚本(安装docker)

``` vagrantfile

config.vm.box = "centos-7" #名字需要与之前定义的vagrant box名称一致
------
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
[docker-pull-tls-handshake-timeout]:https://blog.kelu.org/tech/2017/02/08/docker-pull-tls-handshake-timeout.html

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
## 构建一个简单的docker image
1.创建dockfile
``` dockfile
FROM scratch
ADD hello /
CMD ["/hello"]
```
2.编译
``` shell
docker build -t soderberg/hello-world .
```
3.运行container
``` shell
docker run soderberg/hello-world
```
4.查看docker镜像的层
```shell
docker history 928b474fce0f #=>928b474fce0f 对应的是IMAGE ID
```


