---
layout: post
title:  Vagrant之多节点主机配置
date:   2019-01-03 21:00:48
categories: technology
tags: [Vagrant,虚拟机]
---

修改vagrantfile配置文件来达到目的:

1.第一种方式
``` vagrantfile
Vagrant.configure("2") do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://vagrantcloud.com/search.
  config.vm.box = "centos-7"
  config.vm.define "development" do |development|
		development.vm.network "private_network",ip:"192.168.2.100"
		development.vm.hostname="soderberg-dev"
		development.vm.synced_folder "development","/vagrant"
  end
  
  config.vm.define "production" do |production|
		production.vm.network "private_network",ip:"192.168.2.101"
		production.vm.hostname="soderberg-prod"
		production.vm.synced_folder "production","/vagrant"
  end
```
2.第二种方式
``` vagrantfile
# -*- mode: ruby -*-
# vi: set ft=ruby :

boxes = [
    {
        :name => "development",
        :hostname=>"node-1",
        :eth1 => "192.168.205.10",
        :mem => "1024",
        :cpu => "1"
    },
    {
        :name => "production",
        :hostname=>"node-2",
        :eth1 => "192.168.205.11",
        :mem => "1024",
        :cpu => "1"
    }
]

Vagrant.configure(2) do |config|

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

end
```

> 参考来源
[使用Vagrant管理虚拟机](https://blog.csdn.net/hongweiit/article/details/81148357)

