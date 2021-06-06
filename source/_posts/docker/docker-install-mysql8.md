---
title: docker安装mysql8
comments: true
mp3: 'http://link.hhtjim.com/163/425570952.mp3'
cover: /img/cover/docker/docker-install-mysql8.jpeg
tags:
  - docker
  - mysql
keywords:
  - docker
  - mysql
date: 2020-05-30 13:16:44
updated: 2020-05-30 13:16:44
categories:
---


## docker install mysql8
```
docker run -p 3306:3306 --name mysql8 \
-v $HOME/docker/mysql/conf:/etc/mysql/conf.d \
-v $HOME/docker/mysql/logs:/logs \
-v $HOME/docker/mysql/data:/var/lib/mysql \
--privileged=true --restart always \
-e MYSQL_ROOT_PASSWORD=123456 -d mysql
```