---
layout: post
title:  Docker Compose 的使用
date:   2019-01-28 17:08:48 +0800
categories: technology
tags: [Docker,Docker compose]
---
### 一、安装Docker Compose

仅Linux 平台，windows和mac不需要额外安装该组件
1.获取安装包

```shell
sudo curl -L "https://github.com/docker/compose/releases/download/1.23.2/docker-compose-$(un
ame -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```
2.为docker compose应用执行权限
 ``` shell
 sudo chmod +x /usr/local/bin/docker-compose
 ``` 
3.查看安装是否成功
``` shell
docker-compose --version
```
### 二、Docker compose 相关
1.
