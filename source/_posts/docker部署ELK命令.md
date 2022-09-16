---
title: docker部署ELK
date: 2022-09-14 18:00:05
tags: [docker]
categories: docker
---
# Linux命令

### 防火墙命令

<!--开启防火墙-->
systemctl start firewalld
<!--关闭防火墙-->
systemctl stop firewalld
<!--查看防火墙状态-->
systemctl status firewalld 
<!--开放某个端口-->
firewall-cmd --zone=public --add-port=5045/tcp --permanent 
<!--重新加载配置-->
firewall-cmd --reload
<!--查看已经开放的端口-->
firewall-cmd --zone=public --list-ports

### 删除挖矿病毒

1. top查看占有率
2. ll /proc/pid(病毒pid)
3. rm -rf /boot/grub2/fonts/0wrasx/kthreaddk \(deleted\)
4. crontab -e  
5. dd
6. reboot

# docker部署

### 1、docker部署elasticsearch 

```shell
docker pull elasticsearch:7.14.0
mkdir -p /gdgx/ymk/elk/es/{config,data,logs}
chown -R 1000:1000 /gdgx/ymk/elk
cd /gdgx/ymk/elk/es/config
touch elasticsearch.yml
-----------------------配置内容----------------------------------
cluster.name: "my-es"
network.host: 0.0.0.0
http.port: 9200
-----------------------启动命令--------------------------------
docker run -it  -d -p 4200:9200 -p 4000:9300 --name kes -e ES_JAVA_OPTS="-Xms1g -Xmx1g" -e "discovery.type=single-node" --restart=always -v /gdgx/ymk/elk/es/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml -v /gdgx/ymk/elk/es/data:/usr/share/elasticsearch/data -v /gdgx/ymk/elk/es/logs:/usr/share/elasticsearch/logs elasticsearch:7.14.0

```

### 2、docker 部署kibana

```
docker pull kibana:7.14.0
docker inspect --format '{{ .NetworkSettings.IPAddress }}' es
mkdir -p /gdgx/ymk/elk/kibana/
vim /gdgx/ymk/elk/kibana/kibana.yml
-------------配置信息---------------
server.name: kibana
server.host: "0"
elasticsearch.hosts: ["http://172.17.0.5:9200"]
xpack.monitoring.ui.container.elasticsearch.enabled: true
i18n.locale: "zh-CN"
--------------启动命令--------------------

docker run -d --restart=always --log-driver json-file --log-opt max-size=100m --log-opt max-file=2 --name kibana -p 4001:5601 -v /gdgx/ymk/elk/kibana/kibana.yml:/usr/share/kibana/config/kibana.yml kibana:7.14.0
```

### 3、部署logstash日志过滤、转换工具

```shell
docker pull logstash:7.14.0
docker run -d --name=logstash logstash:7.14.0
docker logs -f logstash
docker cp logstash:/usr/share/logstash /docker/elk/
mkdir /docker/elk/logstash
chmod 777 -R /docker/elk/logstash
----------yml配置信息——--------------------------
vi /docker/elk/logstash/config/logstash.yml
http.host: "0.0.0.0"
xpack.monitoring.elasticsearch.hosts: ["http://114.55.238.234:9200"]
path.config: /usr/share/logstash/config/*.conf
path.logs: /usr/share/logstash/logs
-----------配置信息——------------
vi /data/elk/logstash/config/logstash.conf
# Sample Logstash configuration for creating a simple
# Beats -> Logstash -> Elasticsearch pipeline.
input {
  tcp {
    port => 5045
    type => syslog
  }
 beats {
   port => "5044"
   type => file
  }
}

output {
 stdout { codec => rubydebug }
  
  if[type] == "file" {
 elasticsearch{
   hosts => ["http://114.55.238.234:9200"]
    index => "filelogs-%{+YYYY.MM.dd}"
  }
}
  elasticsearch{
   hosts => ["http://114.55.238.234:9200"]
    index => "mylogs-%{+YYYY.MM.dd}"
  }
}



docker rm -f logstash
-----------启动命令----------
docker run -d \
  --name=logstash \
  --restart=always \
  -p 5044:5044 \
  -v /docker/elk/logstash:/usr/share/logstash \
  -v /var/log/messages:/var/log/messages \
  logstash:7.14.0

```

### 4、docker安装Filebeat



```shell
docker pull docker.elastic.co/beats/filebeat:7.14.0

---------修改yml配置文件----------
vim /docker/elk/filebeat/filebeat.yml

---------配置信息-----------------
filebeat.inputs:
- type: log
  enabled: true
  paths:
    - /home/log/*.log

  multiline.pattern: '^\['
  multiline.negate: true
  multiline.match: after

output.logstash:
  hosts: ["114.55.238.234:5044"]

----------启动命令---------------
docker run --name filebeat -d \
    -v /home/logs:/home/logs:ro \
    -v /docker/elk/filebeat/filebeat.yml:/usr/share/filebeat/filebeat.yml \
    docker.elastic.co/beats/filebeat:7.14.0
```



