---
layout: post
title:  Docker swarm 常用命令
date:   2019-02-18 09:08:48 +0800
categories: technology
tags: [Docker 集群,Docker swarm]
---

Docker有多种方式用来建集群，这里我们采用其内置的`docker swarm`来看看其实现。
集群环境可以通过我们前面的vagrant环境/Linux主机运行`docker-machine create hostname`/[play with docker](https://labs.play-with-docker.com)三种任选其一来搭建。

1.这里我采用了vagrant方式加本机virtualbox建立了3台虚拟机，vagrant文件内容如下：

``` vagrant file
# -*- mode: ruby -*-
# vi: set ft=ruby :
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
2. `setup.sh`脚本用来安装docker以及常用扩展

 ``` setup.sh
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