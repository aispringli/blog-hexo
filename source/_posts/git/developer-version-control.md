---
title: 软件开发版本控制
comments: true
mp3: 'http://link.hhtjim.com/163/425570952.mp3'
cover: /img/cover/git/developer-version-control.jpeg
date: 2020-09-17 21:49:38
updated: 2020-09-17 21:49:38
tags:
categories:
keywords:
---


## 在软件开发过程中主要分以下几个版本历程

| 版本        | 环境  | 说明           |
| ------------- | ------- | ---------------- |
| developer     | alpha   | 最基本的开发环境 |
| release_xxx   | beta    | 预发布的新版本 |
| staging       | staging | 模拟线上测试 |
| master        | mater   | 灰度测试     |
| vxxx.xxx(tag) | pro     | 生产环境     |
| fix-xxx       |         | 修复bug        |
| feature/xxx   |         | 新特性的分支 |

版本迭代历程 \
developer -> release_xxx -> staging -> master -> vxxx.xxx(tag)

结合 Gitlab 的ci，可以便捷的控制代码的分支对应不同的环境，实现代码提交后自动编译发布到对应的环境

``` yml
image: docker

stages:
  - build
  - deploy


variables: &VARIABLES
  BRANCH_NAME: $CI_COMMIT_REF_NAME$CI_MERGE_REQUEST_TARGET_BRANCH_NAME

build:
  stage: build

  rules:
    - if: $CI_COMMIT_TAG
      when: never
    - if: '$CI_COMMIT_BRANCH =~ /^release/'
      when: never
    - if: $CI_COMMIT_BRANCH

  script:
    - docker build -t "$CI_REGISTRY_IMAGE:$BRANCH_NAME-$CI_COMMIT_SHORT_SHA" .
    - docker login -u "$CI_REGISTRY_USER" -p "$CI_REGISTRY_PASSWORD" $CI_REGISTRY
    - docker push "$CI_REGISTRY_IMAGE:$BRANCH_NAME-$CI_COMMIT_SHORT_SHA"

build_version:
  stage: build

  rules:     
    - if: '$CI_COMMIT_BRANCH !~ /^release/'
      when: never
    - if: $CI_COMMIT_TAG
    - if: $CI_COMMIT_BRANCH
  script:
    - echo ${CI_REGISTRY_IMAGE}:${BRANCH_NAME}
    - docker build -t ${CI_REGISTRY_IMAGE}:${BRANCH_NAME} .
    - docker login -u "$CI_REGISTRY_USER" -p "$CI_REGISTRY_PASSWORD" $CI_REGISTRY
    - docker push ${CI_REGISTRY_IMAGE}:${BRANCH_NAME}

deploy:
  stage: deploy
  only:
    - tags
  when: manual
  script:
    - docker login -u "$CI_REGISTRY_USER" -p "$CI_REGISTRY_PASSWORD" $CI_REGISTRY
    - echo ${CI_REGISTRY_IMAGE}:${CI_COMMIT_REF_NAME}
    - docker push ${CI_REGISTRY_IMAGE}:${CI_COMMIT_REF_NAME}

```