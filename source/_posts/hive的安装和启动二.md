---
title: hive的安装和启动二
date: 2022-08-01 16:04:12
tags: [大数据,hadoop,hive]
---
# hive的安装和启动

hive 的下载地址为：https://archive.apache.org/dist/hive/

1) 操作接口采用类 SQL 语法，提供快速开发的能力（简单、容易上手）
2)  避免了去写 MapReduce，减少开发人员的学习成本。

3)  Hive 的执行延迟比较高，因此 Hive 常用于数据分析，对实时性要求不高的场合。

4)  Hive 优势在于处理大数据，对于处理小数据没有优势，因为 Hive 的执行延迟比较高。（不断地开关JVM虚拟机）

5)  Hive 支持用户自定义函数，用户可以根据自己的需求来实现自己的函数。

<!--more-->
hive+spark


# 如何整合spark和hive
set hive.execution.engine=spark;


## hive的安装

注意：官网下载的Hive3.1.2和Spark3.0.0默认是不兼容的。因为Hive3.1.2支持的Spark版本是2.4.5，


1.需要安装mysql 来存储hive的元数据;

a. 查看是否已安装 Mysqlyum list installed mysql*如果检测出已安装 Mysql 则可以先卸载掉, 然后再进行安装;

b. 安装 Mysql 客户端yum -y install mysql

c. 安装 Mysql 服务器端yum -y install mysql_server

d. 安装 Mysql 开发库yum -y install mysql-devel

e. 配置 Mysql 配置文件设置 utf-8 编码vim /etc/my.cnf , 在 my.cnf 文件中添加 default-character-set=utf8;

f. 启动 Mysql 数据库service mysqld start;

g. 创建 root 密码mysqladmin -u root password 123456

h. 进入 Mysql 数据库mysql -hlocalhost -P3306 -uroot -p123456

i. 进入 Mysql 客户端进行授权grant all privileges on . to 'root'@‘%’ identified by 'test_001' with grant option;

flush privileges;

step3：修改 hive 的配置文件

修改 `hive-env.sh` 文件
```
cd /your_directory/apache-hive-2.3.0-bin/conf
cp hive-env.sh.template hive-env.sh

HADOOP_HOME=/your_directory/hadoop-2.7.5export HIVE_CONF_DIR=/your_directory/apache-hive-2.3.0-bin/conf
```
修改 hive-site.xml 文件
```
修改ConnectionUrl DriverName UserName Password

写入mysql的配置
```
Hive 使用 Mysql 作为元数据存储，需要连接 Mysql 数据库，所以将 mysql-connector-java-5.1.38.jar 这个 jar 包上传到 /your_directory/apache-hive-2.3.0-bin/lib这个目录下, 然后启动 Hive。Hive的安装部署就结束了。
## hive的启动
（1）启动hive
bin/hive



