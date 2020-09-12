---
title: kafka-docker-k8s-install
comments: true
mp3: 'http://link.hhtjim.com/163/425570952.mp3'
cover: /img/welcome-cover.jpeg
keywords:
  - kafka
  - docker
  - k8s
date: 2020-09-12 09:18:26
updated: 2020-09-12 09:18:26
tags:
categories:
---


## 使用 docker 安装 kafka 及 manger
注：本机 ip 192.168.66.201

### 1.kafka 依赖 zookeeper，先安装 zookeeper
``` shell
docker run -d --name zookeeper \
	--publish 2181:2181 \
	--volume /etc/localtime:/etc/localtime \
	--restart=always \
	wurstmeister/zookeeper
``` 

### 2.安装 kfaka
``` shell
docker run -d --name kafka --publish 9092:9092 \
    --link zookeeper \
    --env KAFKA_ZOOKEEPER_CONNECT=192.168.66.201:2181 \
    --env KAFKA_ADVERTISED_HOST_NAME=192.168.66.201 \
    --env KAFKA_ADVERTISED_PORT=9092  \
    --volume /etc/localtime:/etc/localtime \
    --restart=always \
    wurstmeister/kafka
```

### 3.安装 kafka manger
``` shell
docker run -d --name kafka-manager \
    --link zookeeper:zookeeper \
    --link kafka:kafka -p 9001:9000 \
    --restart=always \
    --env ZK_HOSTS=zookeeper:2181 \
    sheepkiller/kafka-manager
```

### 4.验证和测试
直接访问 http://192.168.66.201:9001/ 就可以直接使用了


## 使用 k8s 安装 kafka 及 manger

