---
title: docker安装gitlab
comments: true
mp3: 'http://link.hhtjim.com/163/425570952.mp3'
cover: /img/cover/docker/docker-install-gitlab.jpeg
tags:
  - docker
  - gitlab
keywords:
  - docker
  - gitlab
date: 2020-05-30 19:56:28
updated: 2020-05-30 19:56:28
categories:
---


## docker install gitlab
```
docker run --detach \
  --hostname git.aispring.cloud \
  --publish 8443:443 --publish 88:80 --publish 22:22 \
  --name gitlab \
  --restart always \
  --volume $HOME/docker/gitlab/config:/etc/gitlab \
  --volume $HOME/docker/gitlab/logs:/var/log/gitlab \
  --volume $HOME/docker/gitlab/data:/var/opt/gitlab \
  --privileged=true \
  gitlab/gitlab-ce:latest
```
