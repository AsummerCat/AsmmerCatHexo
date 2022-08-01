---
title: hadoop配置历史服务器及其日志采集汇聚四
date: 2022-08-01 14:39:14
tags: [大数据,hadoop]
---
# 配置历史服务器

为了查看程序(job的任务)的历史运行情况,需要配置下历史服务器

vim mapred-site.xml
```
<!--历史服务端地址(对内)-->
<property>
  <name>mapreduce.jobhistory.address</name>
  <value>ip:10020</value>
</property>
<!--历史服务器web端地址(对外)-->
<property>
  <name>mapreduce.jobhistory.webapp.address</name>
  <value>ip:19888</value>
</property>

```
启动历史服务器
```
mapred --daemon start historyserver
```
查看历史服务器
```
ip:19888/jobhistory
```
<!--more-->

# 日志采集汇聚
所有服务器日志汇聚到HDFS,方便查看程序运行详情

注意开启日志聚集功能,需要重启`NodeManager`,`ResourceManager`,`HistoryServer`

## 配置yarn-site.xml
```
<!--开启日志聚集功能-->
<property>
  <name>yarn.log-aggregation-enable</name>
  <value>true</value>
</property>
<!--设置日志聚集服务器地址(这里的地址是历史服务器的地址)-->
<property>
  <name>yarn.log.server.url</name>
  <value>http://ip:19888/jobhistory/logs</value>
</property>

<!--设置日志保留时间为7天-->
<property>
  <name>yarn.log-aggregation.retain-seconds</name>
  <value>604800</value>
</property>
```

## 关闭`NodeManager`,`ResourceManager`,`HistoryServer`

```
mapred --daemon stop historyserver;

sbin/stop-yarn.sh

```