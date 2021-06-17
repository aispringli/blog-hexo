---
title: github镜像同步gitlab
comments: true
mp3: '/music/blog.mp3'
cover: /img/cover/github-mirror-to-gitlab.jpeg
date: 2020-05-31 11:34:02
updated: 2020-05-31 11:34:02
tags:
    - gitlab
    - git
categories:
keywords:
    - gitlab
    - git
---
## Github镜像同步Gitlab

### Gitlab Import Project
在Gitlab上新建项目，选择从Github导入
{% asset_img gitlab-create-new-project.png %}

### Github Access Token
在 GitHub 的个人中心/ Settings /Developer settings 申请镜像同步的 Token
{% asset_img github-access-token.png %}

### Gitlab Finish Import
在Gitlba导入项目时填写在Github上申请的Token，便可以看到Github上的所有项目，选择需要的导入到Gitlab  
完成之后便可以看见导入的项目了   

### 设置 Mirror
在项目的 Settings/Repository 设置Mirroring repositories 即可
{% asset_img gitlab-mirror-set.png %}