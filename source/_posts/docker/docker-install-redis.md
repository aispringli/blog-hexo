---
title: docker安装redis
comments: true
mp3: '/music/blog.mp3'
cover: /img/cover/docker-install-redis.jpeg
tags:
  - docker
  - redis
keywords:
  - docker
  - redis
date: 2020-05-30 10:26:32
updated: 2020-05-30 10:26:32
categories:
---

## docker install redis
```
docker run -p 6379:6379 \
-v $HOME/docker/redis/data:/data  \
--privileged=true --restart always  --name redis \
-d redis redis-server --appendonly yes 
```