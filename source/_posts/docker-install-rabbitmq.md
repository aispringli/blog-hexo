---
title: docker安装rabbitmq
comments: true
mp3: 'http://link.hhtjim.com/163/425570952.mp3'
cover: /img/cover/docker-install-rabbitmq.jpeg
tags:
  - docker
  - rabbitmq
keywords:
  - docker
  - rabbitmq
date: 2020-05-30 13:16:52
updated: 2020-05-30 13:16:52
categories:
---


## docker install rabbitmq
```
docker stop rabbitmq
docker rm rabbitmq
docker run -d --name rabbitmq \
 -p 5672:5672 -p 15672:15672  \
 -v $HOME/docker/rabbitmq/data:/var/lib/rabbitmq \
 --hostname rabbit.aispring.cloud \
 --restart always \
 -e RABBITMQ_DEFAULT_VHOST=  \
 -e RABBITMQ_DEFAULT_USER=admin \
 -e RABBITMQ_DEFAULT_PASS=admin \
rabbitmq:management
```
