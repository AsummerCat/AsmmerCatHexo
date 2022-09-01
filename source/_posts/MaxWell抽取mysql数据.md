---
title: MaxWell抽取mysql数据
date: 2022-09-01 14:07:20
tags: [大数据,MaxWell]
---

# MaxWell抽取mysql数据
根据mysql的binlog日志 抽取数据->kafka

[官网](https://maxwells-daemon.io/)
## 介绍

Maxwell,是用java编写的mysql变更数据抓取软件,它会实时监控Mysql数据库的数据变更操作(包括insert,update,delete),并将变更数据以json格式发送给kafka等流数据处理平台

<font color='red'>注意v1.31.0及其之后的版本仅支持JDK11以上</font>

## 基本原理
可以理解为MaxWell伪装成了一个从库 从binglog日志中抽取数据传送->kafka
<!--more-->
# 操作

## 1.安装MaxWell,运行
```
curl -sLo - https://github.com/zendesk/maxwell/releases/download/v1.38.0/maxwell-1.38.0.tar.gz
tar -zxvf maxwell-1.38.0.tar.gz

cd maxwell-1.38.0

bin/maxwell --user='maxwell' --password='XXXXXX' --host='127.0.0.1' --producer=stdout
```
## 2.运行命令详解:
```
--config   指定配置文件启动

--user     连接mysql的用户

--password 连接mysql的用户的密码

--host     mysql安装的主机名

--producer 生产者模式 (stdout:控制台 , kafka: kafka集群)
```
## 3.修改配置文件,定制启动Maxwell进程
就是将mysql的参数放入到配置文件中 直接启动

在Maxwell里有个`config.peropertis`的配置文件
```
修改

producer=stdout
host=mysql安装的主机名
user=连接mysql的用户
password=连接mysql的用户的密码

运行:
bin/maxwell --config ./config.properties
```

## 4.开启Mysql的binlog日志
推荐格式`row` 按行读取到MaxWell

```
# /etc/my.cnf

[mysqld]
# maxwell needs binlog_format=row
binlog_format=row
server_id=1 
log-bin=maste
```

## 3.创建一个MaxWell元数据库的库
用来记录MaxWell里需要使用到的一些信息

```
CREATE DATABASE maxwell;
```


# 案例

## 输出到kafka
启动zookeeper和kafka,并且按表名进行分区传入kafka
```
bin/maxwell --user='maxwell' --password='XXXXXX' --host='127.0.0.1' --producer=kafka
--kafka.bootstrap.servers=127.0.0.1:9092 --kafka_topic=maxwell

如果需要在kafka上接收的数据按表进行区分
需要手动创建一个有多个分区的topiic
```
#### 修改config.properties
```
producer=kafka
kafka.bootstrap.servers=127.0.0.1:9092

host=127.0.0.1
user=maxwell
password=XXXXXX
kafka_topic=maxwell

# 修改分区参数

## 根据什么条件分区 database,table,primary_key,column
producer_partition_by=table

## 如果选择的是字段类型 这里需要填入
# producer_partition_columns=对应字段id

# producer_partition_by_fallback=
```

```
运行:
bin/maxwell --config ./config.properties
```


## 监控指定的表输出到kafka或者控制台

```
bin/maxwell --user='maxwell' --password='XXXXXX' --host='127.0.0.1' 
--filter 'exclude: *.*,include:maxwell.test'
--producer=kafka
--kafka.bootstrap.servers=127.0.0.1:9092 
--kafka_topic=maxwell 
```
#### 参数详解
```
--filter 过滤条件
exclude: 排除所有库表
include: 指定某个库表
```


## 监控指定表全量数据输出到控制台,数据初始化
```
在MaxWell的元数据表中bootstrap 中插入库表,然后再执行脚本即可全量数据同步
```

```
mysql> insert into maxwell.bootstrap (database_name, table_name) values ('fooDB', 'barTable');
```




