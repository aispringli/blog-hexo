---
title: docker安装gitlab-runner
comments: true
mp3: '/music/blog.mp3'
cover: /img/cover/docker-install-gitlab-runner.jpeg
tags:
  - docker
  - gitlab
  - gitlab-runner
keywords:
  - docker
  - gitlab
  - gitlab-runner
date: 2020-05-30 19:56:28
updated: 2020-05-30 19:56:28
categories:
---


## docker install gitlab-runner

``` 
docker run -d --name gitlab-runner --restart always \
  -v /mnt/i/data/docker/gitlab-runner/config:/etc/gitlab-runner \
  -v /mnt/i/data/docker/gitlab-runner/app:/opt/app \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v  /mnt/i/data/docker/gitlab-runner/.m2:/root/.m2 \
  -v  /mnt/i/data/docker/gitlab-runner/app:/opt/app \
  gitlab/gitlab-runner 
```

## docker gitlab-runner register

``` 
docker exec -it gitlab-runner gitlab-ci-multi-runner register -n \
  --url http://git.aispring.cloud/  \
  --registration-token xQiyGNUb-Ch95ESsxvop \
  --tag-list=master,develop \
  --description "share" \
  --docker-image timoteoponce/docker-mvn-node \
  --docker-volumes /var/run/docker.sock:/var/run/docker.sock \
  --docker-volumes /d/data/docker/gitlab-runner/.m2:/root/.m2 \
  --docker-volumes /d/data/docker/gitlab-runner/app:/opt/app \
  --executor docker
```
