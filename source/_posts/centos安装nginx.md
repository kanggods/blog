---
title: centos安装nginx
date: 2022-07-01 11:23:10
tags: [centos,nginx]
categories: [centos装机必备,环境]
---
## 安装依赖
```shell
yum -y install gcc zlib zlib-devel pcre-devel openssl openssl-devel
```
## 下载稳定版本
```shell
wget http://nginx.org/download/nginx-1.16.1.tar.gz
```
## 解压
```shell
tar -zxvf nginx-1.16.1.tar.gz
```
## 进入目录
```shell
cd nginx-1.16.1
```
## 配置编译
```shell
./configure --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_ssl_module
```
## 安装
```shell
make && make install
```
## 被安装的目录
```shell
/usr/local/nginx/
```
