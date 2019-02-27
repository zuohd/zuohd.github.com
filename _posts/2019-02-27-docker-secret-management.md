---
layout: post
title:  Docker Secret管理以及使用
date:   2019-02-27 09:08:48 +0800
categories: technology
tags: [Docker,Docker secret]
---
我们看到前面的docker-compose.yml文件暴露了数据库的密码，这显然在实际生产环境中是不合适的，那我们怎么办呢？`docker secret`可以帮到我们。

1.当前目录下新建一个文件，把我们要保护的密码添加到内容中去：

2.创建密钥：

`docker secret create my-pw password` #通过文件

``` shell
echo "admin123"|docker secret create my-pw2 -
299f6cjxmukf5lnor9s1r9bm6` #通过标准输入
```

3.查看docker secret：

`docker secret ls`

4.删除docker secret：

`docker secret rm my-pw2`

5.为service分配secret：

```shell
docker service create --name client --secret my-pw2 busybox -sh -c "wh
ile true;do sleep 3600;done"
```

`docker service ps client`查看容器在哪个机器上然后进入该node机器，通过`docker ps`查看container ID,然后`docker exec -it 4e57 sh`进入该容器运行`ls /run/secrets`,我们会发现存在一个名为'my-pw2'的文件，然后试着终端登录mysql：

``` shell
mysql -u root -p
Enter password:
```

输入'my-pw2'的文件设置的密码可以成功进入。

如果我们使用docker compose文件,又将怎么做呢？

``` shell
version: '3'

services:

  web:
    image: wordpress
    ports:
      - 8080:80
    secrets:
      - my-pw
    environment:
      WORDPRESS_DB_HOST: mysql
      WORDPRESS_DB_PASSWORD_FILE: /run/secrets/my-pw
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
    image: mysql
    secrets:
      - my-pw
    environment:
      MYSQL_ROOT_PASSWORD_FILE: /run/secrets/my-pw
      MYSQL_DATABASE: wordpress
    volumes:
      - mysql-data:/var/lib/mysql
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
secrets:
  my-pw:
    external: true
```

运行`docker stack deploy -c docker-compose.yml  wordpress`方式部署。
