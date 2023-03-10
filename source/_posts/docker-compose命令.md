---
title: docker-compose命令
date: 2022-08-04 14:53:08
tags: [docker-compose,命令]
categories: [docker,部署]
---
## 停掉服务，删除容器，不删除镜像
```shell
docker-compose down
```
## 重启/开始/停止服务
```shell
docker-compose restart/start/stop [服务名称]
```
## 运行某个服务
```shell
docker-compose run [服务名称]
```
## 查看服务中使用的镜像
```shell
docker-compose images [服务名称]
```
## 重新构建（强制删除之前的镜像重新打）并启动
```shell
docker-compose up -d --build --force-recreate
```

## 单机部署kafka
version: '3'

services:
kafka:
image: wurstmeister/kafka:2.13-2.7.0
container_name: "kafka"
ports:
- "9092:9092"
environment:
- TZ=Asia/Shanghai
- KAFKA_ZOOKEEPER_CONNECT=zookeeper:2181
- KAFKA_AUTO_CREATE_TOPICS_ENABLE=true
- KAFKA_ADVERTISED_HOST_NAME=${IP}
- KAFKA_ADVERTISED_PORT=9092
- KAFKA_LISTENERS=PLAINTEXT://:9092
- KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://(要求改的ip):9092
volumes:
- ./kafka/kafka/:/kafka
- /var/run/docker.sock:/var/run/docker.sock
restart: always

zookeeper:
image: wurstmeister/zookeeper
ports:
- "2181:2181"
container_name: "zookeeper"
restart: always
