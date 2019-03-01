---
layout: post
title:  Docker swarm 模式下更新 Service 版本
date:   2019-02-27 09:08:48 +0800
categories: technology
tags: [Docker,Docker swarm]
---

当我们在 Docker Swarm 模式下部署了 Service 之后，要如何更新呢？
1.首先 `docker network ls`确保有已经建立了overlay的网络，新建一个Web 容器：

``` shell
docker service create --name web --publish 8080:5000 --network demo zuohd/python-flask-demo:1
.0
```

2.`docker service ps web`发现服务运行在node2上，我们用`vagrant ssh swarm-worker1`进入后执行`docker ps`发现服务已经运行，再次退出进入机器manager上，显然我们想无中断升级服务需要至少水平扩展两个容器：

``` shell
docker service scale web=2
```

3.打开另一个终端进入node1/node2/node3/任意一台机器刷新服务运行结果：

``` shell

sh -c "while true;curl 127.0.0.1:8080&do sleep 1;done"

```

同时在manager机器将更新服务版本到2.0：

``` shell
docker service update --image zuohd/python-flask-demo:2.0 web
```

可以看到另一个终端的输出会切到2.0版本。

4.`docker service update`可以更新许多参数，如更新服务端口：

``` shell
docker service update --publish-rm 8080:5000 --publish-add 8088;5000 web
```

>如若我们想通过docker compose 更新服务，只需要修改相应的docker-compose文件，然后重新运行`docker stack deploy -c docker-compose.yml  web` 即可。