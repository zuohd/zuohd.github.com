---
layout: post
title:  Docker Stack命令使用
date:   2019-02-26 09:08:48 +0800
categories: technology
tags: [Docker,Docker swarm]
---
这里我们使用`docker stack`来部署docker swarm集群的服务，首先我们还是可以利用docker-compose.yml文件去编排需要安装部署的服务。还是以wordpress为例，

``` yaml
version: '3'

services:

  web:
    image: wordpress
    ports:
      - 8090:80 #map port 80 to 8090
    environment:
      WORDPRESS_DB_HOST: mysql
      WORDPRESS_DB_PASSWORD: root
    networks:
      - my-network
    depends_on:
      - mysql
    deploy:
      mode: replicated
      replicas: 3
      restart_policy:
        condition: on-failure
        delay: 5s
        max_attempts: 3
      update_config:
        parallelism: 1
        delay: 10s

  mysql:
    image: mysql:5.7
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: wordpress
    volumes:
      - mysql-data:/var/lib/mysql57
    networks:
      - my-network
    deploy:
      mode: global
      placement:
        constraints:
          - node.role == manager

volumes:
  mysql-data:

networks:
  my-network:
    driver: overlay
```

1.部署wordpress且指定compose file：

`docker stack deploy wordpress -c docker-compose.yml`

2.查询docker swarm的stacks：

 `docker stack ls`

3.查看stack应用的所有任务：

`docker stack ps wordpress`

4.列出一个stack应用的所有服务：

`docker stack sevices wordpress`

5.删除stack：

`docker stack rm wordpress`

浏览器运行`http://192.168.205.10:8090` 可打开安装界面表示部署成功。