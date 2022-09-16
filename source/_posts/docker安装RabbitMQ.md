---
title: docker安装RabbitMQ
date: 2022-07-01 11:49:55
tags: [docker,RabbitMQ]
categories: docker
---
```shell
docker run --name rabbitmq -p 5672:5672 -p 15672:15672 -d --restart=always rabbitmq:management
```
