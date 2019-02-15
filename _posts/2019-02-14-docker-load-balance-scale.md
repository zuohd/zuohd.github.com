---
layout: post
title:  Docker 容器负载均衡
date:   2019-02-14 09:08:48 +0800
categories: technology
tags: [Docker,Docker load balance]
---
前面我们讲述了[Docker Compose 的使用](https://zuohd.github.io/technology/2019/01/28/docker-compose-using.html),这一次我们通过`docker-compose up --scale help`命令查看如何通过scale参数实现负载均衡。

1.增加超时响应时间至120秒，扩展web容器为3个同时运行

``` shell
COMPOSE_HTTP_TIMEOUT=120 docker-compose up --scale web=3 -d

```
2.docker-compose加入负载均衡器镜像

``` docker-compose.yml
version: "3"
services:
  redis:
    image: redis

  web:
    build:
      context: .
      dockerfile: Dockerfile
    environment:
      REDIS_HOST: redis

  lb:
    image: dockercloud/haproxy
    links:
      - web
    ports:
      - 8080:80
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock

```
3.修改app.py为监听80端口
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
    return 'Hello Docker World! I have been seen %s times and hostname is %s.\n' % (redis.get('hits'),socket.gethostname())

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=80, debug=True)

```
4.修改Dockerfile暴露80端口

``` Dockerfile
FROM python:2.7
LABEL maintainer="soderberg zuo<zuo.houde@gmail.com>"
COPY . /app
WORKDIR /app
RUN pip install flask redis
EXPOSE 80                     
CMD [ "python", "app.py" ]

```
5.启动容器

``` shell
docker-compose up --scale web=3 -d
```
6.查看结果我们会发现每次输出不同的web主机名

``` shell
curl 127.0.0.1:8080 #输出 Hello Docker World! I have been seen 14 times and hostname is 586a60052170.

curl 127.0.0.1:8080 #输出 Hello Docker World! I have been seen 15 times and hostname is aa4cc7c45414.

curl 127.0.0.1:8080 #输出 Hello Docker World! I have been seen 16 times and hostname is df8148e9562a.

```

我们也可以通过脚本轮询查看输出的结果：

```shell
[root@localhost lb-scale]# for i in `seq 10`;do curl 127.0.0.1:8080;done
Hello Docker World! I have been seen 1 times and hostname is c3a300e25680.
Hello Docker World! I have been seen 2 times and hostname is d476a12585a4.
Hello Docker World! I have been seen 3 times and hostname is c2d3cb4c54aa.
Hello Docker World! I have been seen 4 times and hostname is c3a300e25680.
Hello Docker World! I have been seen 5 times and hostname is d476a12585a4.
Hello Docker World! I have been seen 6 times and hostname is c2d3cb4c54aa.
Hello Docker World! I have been seen 7 times and hostname is c3a300e25680.
Hello Docker World! I have been seen 8 times and hostname is d476a12585a4.
Hello Docker World! I have been seen 9 times and hostname is c2d3cb4c54aa.
Hello Docker World! I have been seen 10 times and hostname is c3a300e25680.

```
如果我们想减少服务运行个数可以重新执行docker-compose 命令,将web服务个数设置为1

 ``` shell
 docker-compose up --scale web=1 -d

 ```
 此时执行`docker-compose ps`查看服务个数发现web已经缩减：

 ``` output
      Name                  Command            State             Ports          
--------------------------------------------------------------------------------
lb-scale_lb_1      /sbin/tini --               Up      1936/tcp, 443/tcp,       
                   dockercloud- ...                    0.0.0.0:8080->80/tcp     
lb-scale_redis_1   docker-entrypoint.sh        Up      6379/tcp                 
                   redis ...                                                    
lb-scale_web_1     python app.py               Up      80/tcp  
 ```
 