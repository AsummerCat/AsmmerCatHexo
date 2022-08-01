---
title: HDFS读写流程
date: 2022-08-01 14:42:39
tags: [大数据,hadoop,hdfs]
---
![](/img/2022-08-01/18.png)
<!--more-->
![](/img/2022-08-01/19.png)
#  NameNode工作机制
![](/img/2022-08-01/20.png)
Secondary NameNode 会根据检查点 (默认一小时) 询问NameNode 是否需要合并镜像数据和当前操作记录
![](/img/2022-08-01/21.png)
### 检查点的设置
```
hdfs-default.xml

<property>
  <name>dfs.namenode.checkpoint.period</name>
  <value>3600s</value>
  </property>
```
### 一分钟检查一次操作次数,当操作次数达到100w时候,Secondary NameNode执行一次
```
hdfs-default.xml

<property>
  <name>dfs.namenode.checkpoint.ixns</name>
  <value>1000000</value>
  <decription>操作动作次数</decription>
</property>
<property>
  <name>dfs.namenode.checkpoint.check.priod</name>
  <value>60s</value>
  <decription>一分钟检查一次操作次数</decription>
 </property>  
 
```
# DataNode工作机制
![](/img/2022-08-01/22.png)
### 设置上报块时间和扫描块信息时间配置
```
hdfs-default.xml

设置DataNode向NameNode上报块时间,默认6小时
<property>
  <name>dfs.blockreport.intervalMsec</name>
  <value>21600000</value>
  <decription>设置DataNode向NameNode上报块时间</decription>
</property>

设置DataNode扫描自己块信息列表额度时间,默认6小时
<property>
  <name>dfs.datanode.directoryscan.interval</name>
  <value>21600s</value>
  <decription>设置DataNode扫描自己块信息列表额度时间,默认6小时</decription>
 </property>  
```
### DataNode掉线参数设置
默认10分钟+30s 因为3秒一次心跳发送
```
超时时长的计算公式:
    
    TimeOut= 2* dfs.namenode.heartbeat.recheck-interval+ 10* dfs.heartbeat.interval
```
```
hdfs-default.xml

<property>
  <name>dfs.namenode.directoheartbeat.recheck-interval</name>
  <value>300000</value>
  <decription> 默认5分钟 ,超时时间,单位为毫秒</decription>
 </property> 
 
 <property>
  <name>dfs.heartbeat.interval</name>
  <value>3</value>
  <decription>心跳时间 /单位秒</decription>
 </property> 

```
