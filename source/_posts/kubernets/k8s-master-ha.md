---
title: k8s master ha 高可用集群
comments: true
mp3: 'http://link.hhtjim.com/163/425570952.mp3'
cover: https://resource.aispring.cloud/img/jpg/200.jpg
tags:
  - k8s
categories:
  - k8s
keywords:
  - k8s
  - master
  - ha
date: 2020-11-07 22:46:12
updated: 2020-11-07 22:46:12
---

## 前言
在上一节k8s搭建成功后开源使用默认的join添加多node节点，而k8s默认已经对node高可用了，一个node不可用后会自动将应用调度到其他node节点上运行，而且在运行程序时可以设置多个replicas提高可用性，那么问题来了这个时候只有一个master节点，如果master节点挂掉后怎么办?

### master不可用后结果
无法调度node，即此时如果某个pod无法正常工作后无法调度在其他node上启动新的程序
DNS不可用，内置的Core DNS不可用，node间通过服务名调用的程序都会出现异常
node节点上的服务正常运行，master不可用后不会对node上正在运行的程序有任何影响，均可对外正常提供服务

### 如何master高可用
1.k8s集群要有多个master节点
2.master节点数据共享并一致
3.node节点连接多个master节点

## 多master

master主节点初始化时设置自动共享密钥
```
 kubeadm init --control-plane-endpoint "192.168.66.210:6443" --pod-network-cidr=10.11.0.0/16 --upload-certs
```
其他master节点join时加参数control-plane，表示改节点为master节点
```
kubeadm join 192.168.66.210:6443 --token 7jj2yd.rnykldwd9ko8cwcy \
    --discovery-token-ca-cert-hash sha256:bbfb36662a38dff499a94f53681d8f079e1efcc7f3847f673b3043a8b31f7c2e \
    --control-plane --certificate-key 3f968fd729b672f1b956f1d49de293c55afeb3d1478788ffc1bd3bec3a7585a7
```
这样即可看到多个master节点
```
$ kubenet get nodes
```
其他node节点继续join
```
kubeadm join 192.168.66.210:6443 --token 7jj2yd.rnykldwd9ko8cwcy \
    --discovery-token-ca-cert-hash sha256:bbfb36662a38dff499a94f53681d8f079e1efcc7f3847f673b3043a8b31f7c2e \
    --certificate-key 3f968fd729b672f1b956f1d49de293c55afeb3d1478788ffc1bd3bec3a7585a7
```
这样便会发现一个问题，node节点连接的是第一个master节点的IP，第一个master节点不可用后不会自动连接第二个master节点   
所以需要使用Keeplive虚拟IP来解决这个问题

## Keeplive虚拟IP

如何使用Keeplive来解决上面的问题?
分别在master节点上安装 keeplive，并且虚拟一个共同的虚拟ip-192.168.66.210，   
keeplive会自动根据每个节点的权重决定哪个节点是启用的，node节点连接的也是这个虚拟ip，   
当该节点不可用后keeplive会自动在其他节点上启用同样的虚拟ip，这样node节点可以继续连接master节点
但这样有一个问题，那就是永远都是一个master节点在工作，其他master节点一直处于待工作的状态，无法实现负载均衡。


## Nginx负载均衡
每个master节点都安装一个nginx，指向其他master节点，起到负载均衡的作用