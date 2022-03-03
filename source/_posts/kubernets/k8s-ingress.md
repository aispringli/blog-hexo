---
title: k8s的ingress使用
comments: true
mp3: /music/blog.mp3
keywords:
  - k8s
  - ingress
date: 2022-03-03 14:21:23
updated: 2022-03-03 14:21:23
tags:
categories:
---


## 1、安装Ingress
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.1.1/deploy/static/provider/cloud/deploy.yaml
```

## 2、全局开启real ip
修改配置里data的参数
```shell
kubectl -n ingress-nginx edit configmap ingress-nginx-controller
```
```yml
apiVersion: v1
data:
  allow-snippet-annotations: "true"
  compute-full-forwarded-for: "true"
  forwarded-for-header: X-Forwarded-For
  use-forwarded-headers: "true"
kind: ConfigMap
```
## 3、修改body大小限制

```
nginx.ingress.kubernetes.io/proxy-body-size: "100M"
```


## 3、修改隐私限制

```
nginx.ingress.kubernetes.io/server-snippet: |
      add_header Content-Security-Policy "upgrade-insecure-requests;connect-src *";
```