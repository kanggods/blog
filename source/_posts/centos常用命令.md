---
title: centos常用命令
date: 2022-07-01 11:39:11
tags: [centos,命令]
categories: 命令
---
## 开启防火墙
```shell
systemctl start firewalld
```
## 关闭防火墙
```shell
systemctl stop firewalld
```
## 查看防火墙状态
```shell
systemctl status firewalld 
```
## 开放某个端口
```shell
firewall-cmd --zone=public --add-port=80/tcp --permanent
```
## 重新加载配置
```shell
firewall-cmd --reload
```
## 查看已经开放的端口
```shell
firewall-cmd --zone=public --list-ports
```
## 查找（在**中填写你要的查找的路径）
```shell
find / -name **
```
