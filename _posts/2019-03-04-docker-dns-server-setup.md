---
layout: post
title:  搭建 Docker DNS服务器
date:   2019-03-04 07:08:48 +0800
categories: technology
tags: [Devops,CI]
---
如果我们要让 gitlab ci runner 使用docker容器，面临的问题是docker容器如何找到gitlab的DNS，所以我们选择一台新主机做DNS服务，配置步骤如下：

1.`vagrant ssh swarm-worker1`进入 node2 主机

``` shell
docker run -d -p 53:53/tcp -p 53:53/udp --cap-add=NET_ADMIN --name dns andyshinn/dnsmasq
```

2.进入容器开始配置

``` shell
docker exec -it dns sh
vi /etc/resolv.conf   # Add nameserver 8.8.8.8
vi /etc/dnsmasq.hosts # Add 192.168.2.100 gitlab.example.com
echo "resolv-file=/etc/resolv.conf" >> /etc/dnsmasq.conf
echo "addn-hosts=/etc/dnsmasq.hosts" >> /etc/dnsmasq.conf
```

3.回到宿主机重启容器

``` shell
docker restart dns
```

4.进入gitlab-ci的node3 主机，执行

``` shell
vi /etc/resolv.conf #Add nameserver 192.168.205.11,it seems just keep it so that it works!!
```

此处的 IP 是运行DNS服务容器的宿主机 IP，执行`ping gitlab.example.com`测试，如果成功说明 dns 服务配置成功。

5.注册两个 runner，一个针对python2.7，一个针对python3.4,自行修改 token 值

``` shell

sudo gitlab-runner register \
  --non-interactive \
  --url "http://gitlab.example.com/" \
  --registration-token "PROJECT_REGISTRATION_TOKEN" \
  --executor "docker" \
  --docker-image python:2.7 \
  --description "gitlab-ci" \
  --tag-list "python2.7" \
  --run-untagged \
  --locked="false" \

  sudo gitlab-runner register \
  --non-interactive \
  --url "http://gitlab.example.com/" \
  --registration-token "PROJECT_REGISTRATION_TOKEN" \
  --executor "docker" \
  --docker-image python:3.4 \
  --description "gitlab-ci" \
  --tag-list "python3.4" \
  --run-untagged \
  --locked="false" \
```

最后用命令验证 `sudo gitlab-ci-multi-runner verify` 查看 runner 是否 alive，以下是输出结果：

``` shell
Running in system-mode.

Verifying runner... is alive                        runner=dzbz1D7t
Verifying runner... is alive                        runner=Vi7MnTbK
Verifying runner... is alive                        runner=M1kW56Sn
```

好了，现在我们在gitlab项目上添加`.gitlab-ci.yml` 文件：

``` yaml

stages:
  - style
  - test
  - deploy
  
pep8:
  stage: style
  script:
    - pip install tox -i https://pypi.tuna.tsinghua.edu.cn/simple
    - tox -e pep8
  tags:
    - python2.7

unittest-py27:
  stage: test
  script:
    - pip install tox -i https://pypi.tuna.tsinghua.edu.cn/simple
    - tox -e py27
  tags:
    - python2.7

unittest-py34:
  stage: test
  script:
    - pip install tox -i https://pypi.tuna.tsinghua.edu.cn/simple
    - tox -e py34
  tags:
    - python3.4

docker-deploy:
  stage: deploy
  script:
    - docker build -t flask-demo .
    - if [ $(docker ps -aq --filter name=web) ]; then docker rm -f web;fi
    - docker run -d -p 5000:5000 --name web flask-demo
  tags:
    - test  #test是shell类型的runner
  only:
    - master #只有master分支变化，才做CD部署操作

```
