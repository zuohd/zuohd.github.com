---
layout: post
title:  产品发布版本管理
date:   2019-03-05 00:19:48 +0800
categories: technology
tags: [Devops,Production Release]
---

按照 gitflow的流程，任何人不能随意改动 master 分支，进入 project》settings>repository>proteted Branches 更改为 NO ONE，然后设置 project》settings>general>Merge Request,选择“Only allow merge requests to be merged if the pipeline succeeds”。以后我们增加功能都在其他分支开发，如 dev 分支，push 之后再进入 gitlab 选择 “creat merge request”创建一个合并请求。

产品发布新的版本时，我们希望在为 master 分支打个 release tag，然后将最新的 image 推送到内部 docker registry。而生产环境从 docker registry 获取最新的镜像，手动或自动的实现 service 更新。

# 搭建私有的 Docker registry

1.参照[搭建 Docker DNS服务器](https://zuohd.github.io/technology/2019/03/03/docker-dns-server-setup.html)将 registry.example.com 映射到本地host 主机，host主机运行`docker run -d -v /opt/registry:/var/lib/registry -p 5000:5000 --restart=always --name registry registry:2`即可。

2.为了使用docker私有镜像源，我们需要增加一个daemon.json文件

``` shell
vim  /etc/docker/daemon.json # Add {"insecure-registries":["registry.example.com:5000"]}
systemctl daemon-reload
systemctl restart docker
```

然后编辑`.gitlab-ci.yml`:

``` yaml
stages:
  - style
  - test
  - deploy
  - release
  
pep8:
  stage: style
  script:
    - pip install tox -i https://pypi.tuna.tsinghua.edu.cn/simple
    - tox -e pep8
  tags:
    - python2.7
  except:
    - tags #tag变化不执行

unittest-py27:
  stage: test
  script:
    - pip install tox -i https://pypi.tuna.tsinghua.edu.cn/simple
    - tox -e py27
  tags:
    - python2.7
  except:
    - tags #tag变化不执行

unittest-py34:
  stage: test
  script:
    - pip install tox -i https://pypi.tuna.tsinghua.edu.cn/simple
    - tox -e py34
  tags:
    - python3.4
  except:
    - tags #tag变化不执行
    
docker-deploy:
  stage: deploy
  script:
    - docker build -t registry.example.com:5000/flask-demo .
    - docker push registry.example.com:5000/flask-demo
    - if [ $(docker ps -aq --filter name=web) ]; then docker rm -f web;fi
    - docker run -d -p 6000:5000 --name web registry.example.com:5000/flask-demo
  tags:
    - demo
  only:
    - master
    
docker-image-release:
  stage: release
  script:
    - docker build -t registry.example.com:5000/flask-demo:$CI_COMMIT_TAG .
    - docker push registry.example.com:5000/flask-demo:$CI_COMMIT_TAG
  tags:
    - demo
  only:
    - tags # 只在tag变化时执行
```

我们可以刷新 http://registry.example.com:5000/v2/flask-demo/tags/list 来验证容器是否构建成功。