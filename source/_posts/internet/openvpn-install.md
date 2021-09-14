---
title: openvpn快速搭建和配置
comments: true
mp3: '/music/blog.mp3'
date: 2021-06-07 11:05:11
updated: 2021-06-07 11:05:11
tags:
categories:
keywords:
---


## 1. 先在服务器上安装openvpn server

参考 https://github.com/aispringli/oepnvpn-install
```
curl -o https://raw.githubusercontent.com/aispringli/oepnvpn-install/master/openvpn-install.sh
chmod +x openvpn-install.sh
./openvpn-install.sh
```
按照提示操作即可

## 2.固定IP
```
vim /etc/openvpn/ipp.txt
```
该文件每行一条记录，以逗号分隔，前面是client名称，后面是IP地址即可
```
win-company,172.16.66.20
```

## 3.内部组网
我们需要在某个路由器上登录vpn的客户端，并且可以使用任意客户端能够访问该路由器下的所有主机（主机不需要登录vpn客户端）  
这个时候就需要使用iroute了  
修改server.conf
```
route 192.168.66.0 255.255.255.0
```
增加客户端配置文件，路径参考 server.conf配置client-config-dir /etc/openvpn/ccd
```
vim /etc/openvpn/client/{client-name}
```
添加如下网关和子网掩码信息
```
iroute 192.168.1.0 255.255.255.0
```

## 4.配置客户端指定网段流量走vpn
服务端
```
vim /etc/openvpn/server.conf
删掉  
push "redirect-gateway def1 bypass-dhcp"
增加
push "route 172.16.66.0 255.255.255.0 vpn_gateway"
push "route 192.168.66.0 255.255.255.0 vpn_gateway"
```
客户端可以在配置文件下增加如下配置
```
route-nopull
route 172.16.66.0 255.255.255.0 vpn_gateway
route 192.168.66.0 255.255.255.0 vpn_gateway
```