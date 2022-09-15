---
title: docker安装MySQL
date: 2022-07-01 11:49:00
tags: [docker,MySQL]
---
## 拉取5.7的镜像
```shell
docker pull mysql:5.7
```
## 启动容器
```shell
docker run --name mysql -p 3307:3306 --restart=always -e MYSQL_ROOT_PASSWORD=DD123456aa -d mysql:5.7
```
## 进入容器
```shell
docker exec -it mysql bash
```
## 登录
```shell
mysql -u root -p
```
## 开启远程连接
```shell
ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY 'DD123456aa';
```
## 刷新
```shell
flush privileges;
```
