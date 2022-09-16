---
title: Hadoop
date: 2022-09-14 18:00:05
tags: [环境部署]
categories: docker
---
## hadoop入门笔记

### 1、hadoop 安装

##### 1 、服务器设置

1、配置服务器别名和访问  vim  /etc/hosts

2、将三台服务器的ip输入 xxx.xxx.xx hadoop1  xxx.xxx.xx hadoop3
											xxx.xxx.xx hadoop2

3、阿里云服务器下本机地址要设置为私有ip（注意）

##### 2 、jdk环境搭建

1、解压jdk

2、配置环境变量

3、使环境变量生效

4、查看java版本是否安装成功

```shell
tar -zxvf /root/jdk-8u161-linux-x64.tar.gz -C ./
vim /etc/profile
JAVA_HOME=/usr/local/java/jdk1.8.0_161
CLASSPATH=$JAVA_HOME/lib/
PATH=$PATH:$JAVA_HOME/bin
export PATH JAVA_HOME CLASSPATH
source /etc/profile
java -version
```

​										

##### 3、hadoop环境搭建

1、解压hadoop

2、配置环境变量

3、使环境变量生效

4、查看hadoop版本是否安装成功

**遇到的坑：ERROR: Invalid HADOOP_HDFS_HOME 需要在系统下添加**

```
[root@duan local]# vim ~/.bashrc
[root@duan local]# source ~/.bash_profile
```

##### 4、务器直接免密码登录

1、ssh-keygen -t rsa 再 .ssh目录下生成公私密钥

2、 ssh-copy-id duan 公钥给duan

##### 5、hadoop集群配置

权能划分：duan nameNode gao secord nameNode zou RM

##### 6、分发脚本

每台机器执行 yum -y install rsync 安装工具

```shell
#!/bin/bash
#1.获取输入参数个数，如果没有参数，直接退出
pcount=$#
if((pcount==0)); then
echo no args;
exit;
fi

#2.获取文件名称
p1=$1
fname=`basename $p1`
echo fname=$fname

#3.获取上级目录到绝对路径
pdir=`cd -P $(dirname $p1); pwd`
echo pdir=$pdir

#4.获取当前用户名称
user=`whoami`

#5.循环（191,192,193三台机器是我的集群ip）
echo ------------------- hadoop$host --------------
rsync -rvl $pdir/$fname $user@zou:$pdir
rsync -rvl $pdir/$fname $user@duan:$pdir
rsync -rvl $pdir/$fname $user@gao:$pdir

```



##### 7、启动停止

start-dfs.sh/stop-dfs.sh

start-yarn.sh/stop-yarn.sh

##### 8、常用端口号

1、HDFS  NameNode 内部通讯端口：8020、9000、9820

2、HDFS  NameNode web端口 ：9870

3、YARN 查看运行情况得端口8088

4、历史服务器：19888

### core-site.xml

```xml
<configuration>
		<!--设置全局参数，指定HDFS上NN地址为master，端口9000-->
        <property>
                <name>fs.default.name</name>
                <value>hdfs://master:9000</value>
        </property>
        <!--指定HDFS执行时的临时目录，指定NameNode、DataNode、JournalNode等存放数据的公共目录。用户也可以自己单独指定这三类节点的目录。以下的/hdfs/tmp目录与文件都是自己创建的 -->
        <property>
                <name>hadoop.tmp.dir</name>
                <value>/usr/hadoop/hadoop-2.7.3/hdfs/tmp</value>
                <description>A base for other tempory directories</description>
        </property>
        <property>
                <name>io.file.buffer.size</name>                                       
                <value>131072</value>
            		<final>4096</final>
            <description>流文件的缓冲区为4K</description>
        </property>
        <property>
                <name>fs.checkpoint.period</name>
                <value>60</value>
        </property>
        <property>
                <name>fs.checkpoint.size</name>
                <value>67108864</value>
        </property>
</configuration>
```

### 2、hadoop hdfs入门操作

