---
title: centos安装docker
date: 2022-07-01 10:57:10
tags: [centos,docker]
categories: centos装机必备
---
## 卸载旧版本
```shell
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```
## 安装配置相关依赖
```shell
sudo yum install -y yum-utils
```
```shell
sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```
## 安装docker引擎
```shell
sudo yum install docker-ce docker-ce-cli containerd.io
```
## 启动docker
```shell
sudo systemctl start docker
```
## 设置开机自启动
```shell
sudo systemctl enable docker
```
## 验证是否正确安装
```shell
sudo docker run hello-world
```
## 配置阿里云镜像加速
```shell
sudo mkdir -p /etc/docker
```
```shell
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://9w9zqgnf.mirror.aliyuncs.com"]
}
EOF
```
```shell
sudo systemctl daemon-reload
```
```shell
sudo systemctl restart docker
```
