---
layout: post
title:  Docker swarm搭建集群环境
date:   2019-02-18 09:08:48 +0800
categories: technology
tags: [Docker,Docker swarm]
---

Docker有多种方式用来建集群，这里我们采用其内置的`docker swarm`来看看其实现。
集群环境可以通过我们前面的vagrant环境~Linux主机运行`docker-machine create hostname`~[play with docker](https://labs.play-with-docker.com)三种任选其一来搭建。


1.这里我采用了vagrant方式加本机virtualbox建立了3台虚拟机，vagrant文件内容如下：

``` vagrantfile
boxes = [
    {
        :name => "swarm-manager",
		:hostname=>"node-1",
        :eth1 => "192.168.205.10",
        :mem => "1024",
        :cpu => "1"
    },
    {
        :name => "swarm-worker1",
		:hostname=>"node-2",
        :eth1 => "192.168.205.11",
        :mem => "1024",
        :cpu => "1"
    },
	{
        :name => "swarm-worker2",
		:hostname=>"node-3",
        :eth1 => "192.168.205.12",
        :mem => "1024",
        :cpu => "1"
    }
]
# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://vagrantcloud.com/search.
  config.vm.box = "centos-7"
  
  boxes.each do |opts|
      config.vm.define opts[:name] do |config|
        config.vm.hostname = opts[:name]
        config.vm.provider "vmware_fusion" do |v|
          v.vmx["memsize"] = opts[:mem]
          v.vmx["numvcpus"] = opts[:cpu]
        end

        config.vm.provider "virtualbox" do |v|
          v.customize ["modifyvm", :id, "--memory", opts[:mem]]
          v.customize ["modifyvm", :id, "--cpus", opts[:cpu]]
        end

        config.vm.network :private_network, ip: opts[:eth1]
		config.vm.hostname = opts[:hostname]
      end
  end

  config.vm.synced_folder "share", "/vagrant" 
  config.vm.provision "shell", privileged: true, path: "./setup.sh"
  end
```

2.setup.sh脚本用来安装docker以及常用扩展

 ``` shell
#/bin/sh

# install some tools
sudo yum install -y git vim gcc glibc-static telnet bridge-utils

# install docker
curl -fsSL https://get.daocloud.io/docker/ -o get-docker.sh
sh get-docker.sh

# start docker service
sudo groupadd docker
sudo gpasswd -a vagrant docker
sudo systemctl start docker

rm -rf get-docker.sh

```
3.`vagrant ssh swarm-manager`进入node1主机执行

```shell
docker swarm init --advertise-addr=192.168.205.10
```
ip地址可以通过`ip a`查得

4.依据上步执行结果进入node2,node3主机依次执行

```shell
[vagrant@node-2 ~]$ docker swarm join --token SWMTKN-1-01q7bjyh8ev7ih3firurqdrx39ne7340s7x32d8pi9h5yv3luf-7gjs27sjue5it
hxp93wubco4l 192.168.205.10:2377
```
5.进入node1主机查看节点状态

```shell
[vagrant@node-1 ~]$ docker node ls
```
输出结果如下说明集群环境搭建成功

``` output
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VER
SION
zi2n2aynpm5fcmrjpjudl33wv *   node-1              Ready               Active              Leader              18.09.2
h3miabfuytrelktkkd0qdnu75     node-2              Ready               Active                                  18.09.2
mf9juazpaek967rdmmsxvq0me     node-3              Ready               Active                                  18.09.2

```