---
title: centos安装maven
date: 2022-08-04 15:22:00
tags: [centos,maven]
categories: centos
---
## 进入目录
```shell
cd /usr/local
```
## 下载maven
```shell
wget https://archive.apache.org/dist/maven/maven-3/3.6.3/binaries/apache-maven-3.6.3-bin.tar.gz
```
## 解压
```shell
tar -xvf apache-maven-3.6.3-bin.tar.gz
```
## 重命名
```shell
mv apache-maven-3.6.3 maven
```

## 配置
```shell
vim /etc/profile
```
## 添加配置
```shell
export PATH=/usr/local/maven/bin:$PATH
```
## 重新加载配置
```shell
source /etc/profile
```
## 验证是否安装成功
```shell
mvn -V
```
## 进入maven
```shell
cd maven
```
## 创建repository目录
```shell
mkdir repository
```
## 修改配置文件
```shell
cd conf/
```
```shell
vim settings.xml
```
## 配置依赖存储路径
```javascript
<localRepository>/usr/local/maven/repository</localRepository>
```
## 配置阿里云镜像加速
```javascript
<mirror>
    <id>nexus-aliyun</id>
    <mirrorOf>central</mirrorOf>
    <name>nexus-aliyun</name>
    <url>http://maven.aliyun.com/nexus/content/groups/public</url>
</mirror>
```

