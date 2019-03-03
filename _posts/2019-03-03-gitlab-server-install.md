---
layout: post
title:  搭建 GitLab 服务器以及 CI 环境
date:   2019-03-03 21:08:48 +0800
categories: technology
tags: [Devops,CI]
---
*内容概览*

* Do not remove this line (it will not be displayed)
{:toc}

# 一、搭建 GitLab 服务器

搭建 GitLab 服务器需要至少4G 内存，所以我们的 vagrant 文件中需要取消以下内容的注释并修改相关的内存值：

``` shell
 config.vm.provider "virtualbox" do |vb|  
     # Customize the amount of memory on the VM:
     vb.memory = "4096"
 end
```

1.参考[Install Gitlab CE on CentOS 7](https://computingforgeeks.com/install-gitlab-ce-on-centos-7-fedora-29-fedora-28/)进入虚拟机,执行以下命令：

``` shell
sudo yum install -y curl policycoreutils-python openssh-server
sudo systemctl enable sshd
sudo systemctl start sshd

sudo yum install postfix
sudo systemctl enable postfix
sudo systemctl start postfix

```

2.新建/etc/yum.repos.d/gitlab-ce.repo,内容为

``` shell
[gitlab-ce]
name=Gitlab CE Repository
baseurl=https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el$releasever/
gpgcheck=0
enabled=1
```

3.安装GitLab且设置域名为 gitlab.example.com (如果是本地域名可以尝试将虚拟机 IP 添加 hosts 文件中)

```shell
sudo yum makecache
sudo EXTERNAL_URL="http://gitlab.example.com" yum install -y gitlab-ce
```

4.配置并启动 GitLab

``` shell
sudo gitlab-ctl reconfigure
```

可以通过`sudo gitlabctl status` 查看 GitLab 进程状态。
本地访问 http://gitlab.example.com ，修改 root 的密码，再次登录后，新建 group 和 project，加入一些文件，用`git clone`看看可否下载 repository。

我们还可以进入 GitLab 服务器执行`sudo vim /etc/gitlab/gitlab.rb`，通过修改其 external_url 值可以更新我们的域名，别忘了最后要执行`sudo gitlab-ctl reconfigure`才可以使得更改生效。

# 二、搭建 CI 服务器

进入另一台 vagrant 虚拟机，我们还需要搭建一个 GitLab CI Multi Runner 环境。

1.`sudo vi /etc/yum.repos.d/gitlab-ci-multi-runner.repo` 新建文件，加入如下内容

``` shell
[gitlab-ci-multi-runner]
name=gitlab-ci-multi-runner
baseurl=https://mirrors.tuna.tsinghua.edu.cn/gitlab-ci-multi-runner/yum/el7
repo_gpgcheck=0
gpgcheck=0
enabled=1
gpgkey=https://packages.gitlab.com/gpg.key
```

2.再执行安装

``` shell
sudo yum makecache
sudo yum install gitlab-ci-multi-runner
```

安装完毕后使用`sudo gitlab-ci-multi-runner status` 查看服务运行状态

3.设置 Docker 权限
为了能让 gitlab-runner 正确的执行 docker 命令，需要把 gitlab-runner 用户添加到 docker group 里，然后重启 docker 和 gitlab ci runner

``` shell
sudo usermod -aG docker gitlab-runner
sudo service docker restart
sudo gitlab-ci-multi-runner restart
```

4.将 gitlab.example.com 域名以及对应的 IP 加入 /etc/hosts

``` shell
sudo vim /etc/hosts
```

5.注册 gitlab-ci-multi-runner

``` shell
sudo gitlab-ci-multi-runner register
```

填入gitlab网址 http://gitlab.example.com/
进入 gitlab.example.com，从 settings-> CI/CD ->Runners 展开，`Set up a specific Runner manually`描述部分有 registration token，将此值填入gitlab token：paVNAZ5Kx1qzs5_hCzyM，tags填入`test,demo`,the executor 填入`shell`,其他默认即可。

6.`sudo gitlab-ci-multi-runner  list`查看已经注册的 runner

``` output
[vagrant@node-3 ~]$ sudo gitlab-ci-multi-runner  list
Listing configured runners                          ConfigFile=/etc/gitlab-runner/config.toml
gitlab-ci                                           Executor=shell Token=dzbz1D7tVMA3yydw8Qav URL=http://gitlab.example.
com/
```

再次进入 http://gitlab.example.com/ ，从 settings-> CI/CD ->Runners 展开，发现 Runners activated for this project，证明已经连接成功。

7.为项目添加 CI 配置，文件名为 `.gitlab-ci.yml`,通常我们的持续集成分3个阶段：构建->测试->部署，所以我们的文件内容如下：

``` yaml
# define stages
stages:
  - build
  - test
  - deploy
# define job  
job1:
  stage: test
  tags:
    - demo #specify the runner tag
  script:
    - echo "I am job1"
    - echo "I am in test stage"
job2:
  stage: build
  tags:
    - demo
  script:
    - echo "I am job2"
    - echo "I am in build stage"
job3:
  stage: deploy
  tags:
    - demo
  script:
    - echo "I am job3"
    - echo "I am in deploy stage"
```

进入菜单 CI/CD->Pipelines，可以查看持续集成部署状态以及每个 job 详情，注意如果前一个job失败，后面的job也不会运行，至此配置完毕。