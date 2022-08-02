---
title: CAT的安装一
date: 2022-08-02 11:02:01
tags: [链路跟踪,cat]
---
# CAT的安装
可以直接github上面编译源码然后打包`war`直接部署

## 优点
聚合报表丰富,中文支持好,国内案例多
大众点评开源

## 缺点
代码侵入性高
<!--more-->

## 模块介绍
```
cat-client : 客户端 上报监控数据

cat-consumer: 服务端,收集监控数据进行统计分析,构建丰富的统计报表

cat-alarm: 实时告警,提供报表指标的监控告警

cat-hadoop: 数据存储,logview存储到HDFS

cat-home: 管理端,报表展示,配置管理等

```

## 支持的报表

| 报表名称 | 报表内容 |
| --- | --- |
| Transaction报表 | 一段代码的运行时间,次数,比如URL/cache/sql执行次数相应时间 |
|Event报表|一段代码运行次数,比如出现一次异常|
|Problem报表|根据Transaction/Event数据分析出系统可能出现的一次,慢程序|
|Heartbeat报表|JVM状态信息|
|Business报表|业务指标等,用户可以自己定制|


## Cat服务端安装
服务登录初始化密码 admin admin
### 根据提供的cat初始化脚本初始化数据库

### 服务器上建立datasources.xml

服务器上建立文件夹 `/data/appdatas/cat/` 并创建`datasources.xml`文件
路径是固定的
```
<?xml version="1.0" encoding="utf-8"?>
<data-sources>
    <data-source id="cat">
        <maximum-pool-size>3</maximum-pool-size>
        <connection-timeout>1s</connection-timeout>
        <idle-timeout>10m</idle-timeout>
        <statement-cache-size>1000</statement-cache-size>
        <properties>
            <driver>com.mysql.jdbc.Driver</driver>
            <url><![CDATA[jdbc:mysql://数据库ip:3306/cat]]></url>  <!-- 请替换为真实数据库URL及Port  -->
            <user>用户名</user>  <!-- 请替换为真实数据库用户名  -->
            <password>密码</password>  <!-- 请替换为真实数据库密码  -->
            <connectionProperties><![CDATA[useUnicode=true&characterEncoding=UTF-8&autoReconnect=true&socketTimeout=120000]]></connectionProperties>
        </properties>
    </data-source>
</data-sources>
```
启动tomcat即可
### 配置服务端相关配置
```
http://127.0.0.1:8080/cat/s/confg?op=serverConifgUpdate
```
### 服务端路由相关配置
```
http://127.0.0.1:8080/cat/s/confg?op=routerConifgUpdate
```

## cat 客户端
### 在cat 客户端 服务器上建立文件夹 `/data/appdatas/cat/` 并创建client.xml文件
```
<?xml version="1.0" encoding="utf-8"?>
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema" xsi:noNamespaceSchemaLocation="config.xsd">
    <servers>
        <server ip="127.0.0.1" port="2280" http-port="8080" />
    </servers>
</config>
```
127.0.0.1 替换为cat 服务端tomcat的ip , 8080替换为服务端tomcat的端口


## 集成项目
```
<dependency>
    <groupId>com.dianping.cat</groupId>
    <artifactId>cat-client</artifactId>
    <version>${cat.version}</version>
</dependency>
```
