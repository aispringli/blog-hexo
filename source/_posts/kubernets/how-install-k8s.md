---
title: Centos8安装K8s
comments: true
mp3: 'http://link.hhtjim.com/163/425570952.mp3'
cover: /img/cover/kubernets/how-install-k8s.jpg
date: 2020-06-25 23:44:22
updated: 2020-06-25 23:44:22
tags:
    - k8s
categories:
    - k8s
keywords:
    - k8s
    - docker
---
## 1、安装 Docker
### 1. 添加依赖源
``` 
$ curl https://download.docker.com/linux/centos/docker-ce.repo -o /etc/yum.repos.d/docker-ce.repo
```
### 2. 安装Docker
``` 
$ yum -y install docker-ce
```
### 3. 设置Docker开机自启动
``` 
$ systemctl enable --now docker
$ systemctl enable docker.service
```
### 4. 设置阿里云镜像源加速
``` 
$ sudo mkdir -p /etc/docker
$ sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://ie2mtbp8.mirror.aliyuncs.com"]
}
EOF
$ sudo systemctl daemon-reload
$ sudo systemctl restart docker
```

## 2、安装 K8s
### 1. 添加依赖源
``` 
$ cat > /etc/yum.repos.d/kubernetes.repo <<EOF
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
EOF
```
### 2. 安装kubeadm
``` 
$ yum install -y kubelet kubeadm kubectl
```
### 3. 设置Kubelet开机自启动
``` 
$ systemctl enable kubelet.service
```
### 4. 下载镜像
在墙外,需要科学上网
``` 
$ kubeadm config images pull
```
### 5. 禁用Swap
``` 
$ vim /etc/fstab
/dev/mapper/cl-swap    >> # /dev/mapper/cl-swap
$ echo vm.swappiness=0 >> /etc/sysctl.conf
```
### 6. Docker设置Systemd
``` 
$  vim /etc/docker/daemon.json
"exec-opts": ["native.cgroupdriver=systemd"]
```
### 7. 初始化
``` 
$ kubeadm init  --pod-network-cidr=10.11.0.0/16
$ mkdir -p $HOME/.kube
$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
$ sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
### 8. 安装 flannel
``` 
$  kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```
### 9. master 允许调度
默认安装好后为master，并且有个taint不允许调度，如需master当作node可去除该taint即可
```
kubectl taint nodes --all node-role.kubernetes.io/master-
```

## 2、K8s集群
### 1. join master
在其他服务器上安装kubeadm，然后安装master上init后的提示kubeadm join....  
``` 
 kubeadm join 192.168.66.211:6443 --token ynw3rv.du4nm0ef6djh3293 --discovery-token-ca-cert-hash sha256:e639aa22ea901ab8956ea40069f36b30df72d9999b42144a610c73fa70485b2e
```
注意：token仅24小时有效，过期后需要重新生成，这里提供一个脚本在过期后一键生成
```
#!/bin/bash

if [ $EUID -ne 0 ];then
    echo "must be root (or sudo) to run this script"
    exit 1
fi

if [ $# != 1 ] ; then
    echo "must input a param master-hostname | master-ip-address"
    echo " e.g.: 192.168.66.66 or k8s.aispring.cloud"
    exit 1;
fi

token=`kubeadm token create`
cert_hash=`openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | openssl dgst -sha256 -hex | sed 's/^.* //'`

echo "Refer the following command to join kubernetes cluster:"
echo "kubeadm join $1:6443 --token ${token} --discovery-token-ca-cert-hash sha256:${cert_hash}"
```
```
./generate_join_master.sh 192.168.66.211
```
