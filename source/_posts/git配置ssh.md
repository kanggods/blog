---
title: git配置ssh
date: 2022-07-01 15:40:04
tags: [Git, SSH]
---
## 配置用户名
```shell
git config --global user.name "yoonada"
```
## 配置邮箱
```shell
git config --global user.email "m15602498163@163.com"
```
## 生成 ssh key
```shell
ssh-keygen -t rsa -b 4096 -C "m15602498163@163.com"
```
## 生成路径
```text
window的生成路径：C:\Users\用户\.ssh
Linux的生成路径：/etc/ssh
```
