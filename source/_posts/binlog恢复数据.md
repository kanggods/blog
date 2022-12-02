---
title: binlog恢复数据
tags: 标签
date: 2022-12-01 15:45:47
categories: [mysql]
tags: [mysql]
---
##### Binlog文件操作

binlog变量查看

```查看
show varibles like 'logbin'
```

binlog功能开启

```
set global log_bin=mylogbin;
```

需要修改my.cnf配置文件

```
binlog-format = ROW //行记录
log_bin = mylogbin // 文件名
```

使用show binlog events 命令查看日志

```
+------------------+-----+----------------+-----------+-------------+---------------------------------------+
| Log_name         | Pos | Event_type     | Server_id | End_log_pos | Info                                  |
+------------------+-----+----------------+-----------+-------------+---------------------------------------+
| mysql-bin.000001 |   4 | Format_desc    |       142 |         123 | Server ver: 5.7.36-log, Binlog ver: 4 |
| mysql-bin.000001 | 123 | Previous_gtids |       142 |         154 |                                       |
+------------------+-----+----------------+-----------+-------------+---------------------------------------+

Previous_gtids:上一个binlog结束的gtid值
Anonymous_Gtid：记录的事务的GTID值
Query：具体的执行语句，具体内容稍后解释。
Rotate:这个类型前面的文章也介绍了，这是binlog文件中的最后一个event，记录下一个binlog的信息。
```

show binlog status 查看当前使用那个库

使用show binlog events in "log_name" 命令查看具体的内容

使用binlog恢复

```
按照时间回复
mysqlbinlog --start-datetime="2020-04" --stop-datetime="2020-05" mysql-bin.000019  | 数据库账号密码
按照位置回复
mysqlbinlog --start-position=154 --stop-position=957 mysql-bin.000019 |
```

删除binlog文件

```
purge binary logs to 文件名
purge binary logs before 日期
reset master;
```

