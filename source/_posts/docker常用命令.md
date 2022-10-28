---
title: docker常用命令
date: 2022-07-01 15:53:36
tags: [docker,命令]
categories: [docker,部署]
---
# docker
## 启动docker服务
```shell
systemctl start docker
```
## 关闭docker服务
```shell
systemctl stop docker
```
## 重启docker服务
```shell
systemctl restart docker
```
## 搜索镜像
```shell
docker search java
```
## 下载镜像
```shell
docker pull java:8
```
## 列出镜像
```shell
docker images
```
## 删除镜像
* 指定名称删除镜像
```shell
docker rmi java:8
```
* 指定名称删除镜像（强制）
```shell
docker rmi -f java:8
```
* 删除所有没有引用的镜像
```shell
docker rmi `docker images | grep none | awk '{print $3}'`
```
* 强制删除所有镜像
```shell
docker rmi -f $(docker images)
```
## 列出容器
* 列出运行中的容器
```shell
docker ps
```
* 列出所有容器
```shell
docker ps -a
```
## 停止容器
```shell
docker stop $ContainerName(or $ContainerId)
```
## 强制停止容器
```shell
docker kill $ContainerName
```
## 启动容器
```shell
docker start $ContainerName
```
## 进入容器
* 先查询出容器的pid
```shell
docker inspect --format "{{.State.Pid}}" $ContainerName
```
* 根据容器的pid进入容器
```shell
nsenter --target "$pid" --mount --uts --ipc --net --pid
```
## 查看容器的IP地址
```shell
docker inspect --format '{{ .NetworkSettings.IPAddress }}' $ContainerName
```
## 将容器启动方式改为always
```shell
# 将容器启动方式改为always
docker container update --restart=always $ContainerName
```
## 同步宿主机时间到容器
```shell
docker cp /etc/localtime $ContainerName:/etc/
```
## 指定容器时区
```shell
docker run -p 80:80 --name nginx \
-e TZ="Asia/Shanghai" \
-d nginx:1.17.0
```
## 查看容器资源占用状况
* 查看指定容器资源占用状况，比如cpu、内存、网络、io状态
```shell
docker stats $ContainerName
```
* 查看所有容器资源占用情况
```shell
docker stats -a
```
## 查看容器磁盘使用情况
```shell
docker system df
```
## 执行容器内部命令
```shell
docker exec -it $ContainerName /bin/bash
```
## 指定账号进入容器内部
```shell
# 使用root账号进入容器内部
docker exec -it --user root $ContainerName /bin/bash
```
## 查看所有网络
```shell
docker network ls
```
## 创建外部网络
```shell
docker network create -d bridge my-bridge-network
```
## 指定容器网络
```shell
docker run -p 80:80 --name nginx \
--network my-bridge-network \
-d nginx:1.17.0
```
## Docker容器清理
* 查看Docker占用的磁盘空间情况
```shell
docker system df
```
* 删除所有关闭的容器
```shell
docker ps -a | grep Exit | cut -d ' ' -f 1 | xargs docker rm
```
* 删除所有dangling镜像(没有Tag的镜像)
```shell
docker rmi $(docker images | grep "^<none>" | awk "{print $3}")
```
* 删除所有dangling数据卷(即无用的 volume)
```shell
docker volume rm $(docker volume ls -qf dangling=true)
```
