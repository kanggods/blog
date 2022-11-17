---
title: nacos集群出现Grpc问题
tags: [nginx]
categories: [日常问题]
date: 2022-09-29 14:13:27
---
### 1、问题产生原因

nacos2.x版本以后引入grpc，端口是默认偏移+1000。原来8848偏移1000就是9848。所以我们要开启两个端口。

### 2、问题解决

1、在nginx.conf文件下加入这行。

stream {
upstream nacosgrpcc {
server 116.62.190.20:9848;
server 47.98.236.213:9848;
server 114.55.238.234:9848;
}
server {
listen 9858;
proxy_connect_timeout 300s;
proxy_timeout 300s;
proxy_pass nacosgrpcc;
}
}

2、安装stream使nginx支持这个模块

yum install -y nginx-mod-stream

3、 nginx -s reload
