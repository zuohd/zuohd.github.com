---
layout: post
title:  Docker Compose 的使用
date:   2019-01-28 17:08:48 +0800
categories: technology
tags: [Docker,Docker compose]
---
### 一、安装Docker Compose

仅Linux 平台，windows和mac不需要额外安装该组件,国内[daocloud][daocloud-link]提供了国内镜像速度稳定且较快。

1.获取安装包并记得修改其中的版本号

```shell
sudo curl -L "https://github.com/docker/compose/releases/download/1.23.2/docker-compose-$(un
ame -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```
``` shell
curl -L https://get.daocloud.io/docker/compose/releases/download/1.23.2/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
```
如果我们安装了pip,也可以通过pip安装
``` shell
pip install docker-compose
```
2.为docker compose应用执行权限
 ``` shell
 sudo chmod +x /usr/local/bin/docker-compose
 ``` 
3.查看安装是否成功
``` shell
sudo docker-compose --version
```
### 二、Docker compose 相关
1.启动容器编排，如果docker-compose.yml没在当前目录，需指定f参数：
```shell
sudo docker-compose up
```
2.docker-compose.yml文件内容如下，具体文件规则请参照[了解YAML][yaml-link]

注意mysql卷目录应该提前建好，且不能跟其他版本mysql目录有冲突，否则会导致启动失败
``` docker-compose.yml
version: '3'
services:
  wordpress:
    image: wordpress
    ports:
      - 8090:80
    environment:
      WORDPRESS_DB_HOST: mysqldb
      WORDPRESS_DB_PASSWORD: wordpress
    networks:
      - my-bridge
  mysqldb:
    image: mysql:5.7
    environment:
      MYSQL_ROOT_PASSWORD: wordpress
      MYSQL_DATABASE: wordpress
    volumes:
      - mysql-data:/var/lib/mysql57
    networks:
      - my-bridge

volumes:
  mysql-data:

networks:
  my-bridge:
    driver: bridge

```
访问127.0.0.1:8090，浏览器会显示wordpress安装界面表示容器启动成功

3.docker-compose 从本地构建镜像,新建一个目录，其下新建3个文件：

~ app.py:

``` python
from flask import Flask
from redis import Redis
import os
import socket
app = Flask(__name__)
redis = Redis(host=os.environ.get('REDIS_HOST', '127.0.0.1'), port=6379)
@app.route('/')
def hello():
    redis.incr('hits')
    return 'Hello Docker World! I have been seen %s times.\n' % (
        redis.get('hits'))

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000, debug=True)

```

~ Dockerfile:

``` Dockerfile
FROM python:2.7
LABEL maintainer="soderberg zuo<zuo.houde@gmail.com>"
COPY . /app
WORKDIR /app
RUN pip install flask redis
EXPOSE 5000
CMD [ "python", "app.py" ]
```
~ docker-compose.yml:

``` docker-compose.yml
version: "3"
services:
  redis:
    image: redis
  web:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - 8080:5000
    environment:
      REDIS_HOST: redis
```

4.docker-compose常用命令
``` shell
docker-compose ps #列出所有的services
docker-compose down #删除docker-compose定义的所有资源并停止相关的service
docker-compose start #启动所有的service
docker-compose stop #终止所有的service
docker-compose up -d #守护进程启动service但不输出日志
docker-compose images #列出所有的容器以及镜像信息
docker-compose exec mysqldb bash #进入mysqldb容器中执行bash命令
docker-compose build #构建镜像
docker-compose build --no-cache web #不带缓存的构建web服务，当我们修改了web程序的内容需要刷新缓存
docker-compose config -q #验证（docker-compose.yml）文件配置，当配置正确时，不输出任何内容，当文件配置错误，输出错误信息。
```
>参考来源

[Docker-compose常用命令](https://www.cnblogs.com/moxiaoan/p/9299404.html)

[daocloud-link]:https://get.daocloud.io/
[yaml-link]:http://docs.saltstack.cn/topics/yaml/index.html