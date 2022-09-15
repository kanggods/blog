---
title: docker-compose命令
date: 2022-08-04 14:53:08
tags: [docker-compose,命令]
categories: 命令
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
