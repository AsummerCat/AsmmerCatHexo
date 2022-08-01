---
title: ETL工具之Sqoop安装一
date: 2022-08-01 14:17:15
tags: [ETL工具,Sqoop,大数据]
---
# ETL工具之Sqoop安装
用于Hadoop生态体系和RDBMS体系之间传送数据的一种工具

```
Hadoop生态系统 : HDFS  Hive  Hbase等

RDBMS体系: mysql oracle db2等

```
Sqoop 可以理解为: sql 到 Hadoop 和 Hadoop 到SQL


### 1.下载解压sqoop
https://sqoop.apache.org

### 2.修改配置文件

1) 重命名配置文件
```
cd conf 
mv sqoop-env-template.sh sqoop-env.sh
```
<!--more-->

2) 修改配置文件
   `sqoop-env.sh`
   环境变量配置进去
```
export HADOOP_COMMON_HOME=hadoop安装位置
export HADOOP_MAPRED_HOME=hadoop安装位置
export HBASE_HOME=habase安装位置
export HIVE_HOME=hive安装位置
export ZOOEEPER_HOME=zookeper安装位置
export ZOOCFGDIR_HOME=zookeper安装位置
```


### 3. 拷贝jdbc的驱动
```
cp jdbc驱动 -> sqoop/lib
```

### 4. 验证sqoop配置是否正确
```
bin/sqoop help
```

### 5. 验证是否能连接上mysql
```
bin/sqoop list-databases --connect jdbc:mysql://192.168.1.1:3306/  --username root --password 123456
```