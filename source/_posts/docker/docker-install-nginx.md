---
title: docker安装nginx
comments: true
mp3: '/music/blog.mp3'
cover: /img/cover/docker-install-nginx.jpeg
tags:
  - docker
  - nginx
keywords:
  - docker
  - nginx
date: 2020-05-30 19:56:28
updated: 2020-05-30 19:56:28
categories:
---


## docker install nginx
```
docker stop nginx
docker rm nginx
docker run -p 80:80 \
 -v /mnt/i/data/docker/nginx/conf.d:/etc/nginx/conf.d  \
 --privileged=true --restart always  --name nginx \
-d nginx
```
## 配置conf.d
在conf.d文件夹下建立自定义conf.d文件，内容如下
```
 server {
    listen       80;
    server_name git.aispring.cloud;
	location / {
        proxy_pass          http://aispring.cloud:88;
        proxy_redirect      off;
        proxy_set_header    Host               $http_host;                      #传递主机名
        proxy_set_header    X-Forwarded-Host   $http_host;                      #传递真实主机名
        proxy_set_header    X-Real-IP          $remote_addr;                    #传递用户IP
        proxy_set_header    X-Forwarded-For    $proxy_add_x_forwarded_for;      #传递用户真实IP
        proxy_set_header    X-Forwarded-Proto  $scheme;                         #传递协议http/https
        proxy_set_header    Upgrade            $http_upgrade;                   #反向代理websocket
        #proxy_set_header    Connection         $connection_upgrade;             #反向代理websocket
        proxy_set_header    Connection         "Upgrade";
        proxy_http_version  1.1;                                                #传递协议版本号
		client_max_body_size  500m;
	}
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}
```
