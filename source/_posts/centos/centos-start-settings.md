---
title: Centos8安装和基本设置
comments: true
mp3: '/music/blog.mp3'
cover: /img/cover/centos-start-settings.jpeg
categories:
  - centos8教程
keywords:
  - centos8
date: 2020-10-24 22:58:10
updated: 2020-10-24 22:58:10
tags:
  centos
---

## 1、安装Centos8
mini 版： http://isoredirect.centos.org/centos/8/isos/x86_64/

## 2、开启SSH
### 1. 修改config
``` 
$ vim /etc/ssh/sshd_config
Port 2222
PermitRootLogin yes
PasswordAuthentication yes
```
### 2. 关闭 SELINUX
不关闭ssh服务无法启动
``` 
$ vim /etc/selinux/config
SELINUX=disabled
```
### 3. 关闭 firewalld
``` 
$ systemctl disable firewalld.service
```

### 4. 开启路由转发
``` 
$ vim /etc/sysctl.conf
net.ipv4.ip_forward = 1
```


