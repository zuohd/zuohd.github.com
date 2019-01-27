---
layout: post
title:  Docker端口映射
date:   2019-01-27 17:49:48 +0800
categories: technology
tags: [DOCKER Port]
---

运行一个nginx容器,命名为`webserver`，并将容器端口80映射到本机
```shell
docker run --name webserver -d -p 80:80 nginx
```
1.查看启动中的容器
``` shell
docker ps -a
```
输出结果查看port可以看到端口映射情况

>CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS
      NAMES
5e62c8c47f18        nginx               "nginx -g 'daemon of…"   12 seconds ago      Up 10 seconds       0.0.0.0:80->80/
tcp   web

2.本地机器测试nginx服务
```shell
curl 127.0.0.1
```
执行结果显示服务成功访问

```html
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```




