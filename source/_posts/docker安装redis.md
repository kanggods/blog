---
title: docker安装redis
date: 2022-07-01 11:49:20
tags: [docker,redis]
---
```shell
docker run --name redis -p 6380:6379 -d --restart=always redis:latest redis-server --appendonly yes --requirepass "DD123456aa"
```
