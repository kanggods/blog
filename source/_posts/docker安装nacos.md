---
title: docker安装nacos
date: 2022-07-01 11:49:11
tags: [docker,nacos]
categories: [docker,部署]
---
## 拉取docker镜像
```shell
docker pull nacos/nacos-server:1.4.2
```
## 创建临时容器（用来拷贝配置、日志使用）
```shell
docker run -p 8848:8848 --name nacostest -d nacos/nacos-server:1.4.2
```
## 创建文件夹
```shell
mkdir -p /mydata/nacos/conf
```
```shell
mkdir -p /mydata/nacos/logs
```
## 配置文件复制
```shell
docker cp nacostest:/home/nacos/logs/ /mydata/nacos/
```
```shell
docker cp nacostest:/home/nacos/conf/ /mydata/nacos/
```
## 删除临时容器
```shell
docker stop nacostest
```
```shell
docker rm nacostest
```
## 创建并启动容器
```shell
docker run -d \
-e MODE=standalone \
-e SPRING_DATASOURCE_PLATFORM=mysql \
-e MYSQL_SERVICE_HOST=43.142.62.156 \
-e MYSQL_SERVICE_PORT=3307 \
-e MYSQL_SERVICE_USER=root \
-e MYSQL_SERVICE_PASSWORD=DD123456aa \
-e MYSQL_SERVICE_DB_NAME=nacos_config \
-e TIME_ZONE='Asia/Shanghai' \
-v /mydata/nacos/logs:/home/nacos/logs \
-v /mydata/nacos/conf:/home/nacos/conf \
-p 8848:8848 \
--name nacos \
--restart=always \
nacos/nacos-server:1.4.2
```
