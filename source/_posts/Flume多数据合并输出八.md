---
title: Flume多数据合并输出八
date: 2022-08-12 09:45:39
tags: [Flume,大数据]
---
# Flume多数据合并输出

流程: 两个数据源聚合到一个sink上输出
分别采集

使用`avro` RPC框架做采集

<!--more-->
# 配置
配置 Source 用于监控 hive.log 文件，配置 Sink 输出数据到下一级 Flume。
### A数据源
```
a1.sources = r1
a1.sinks = k1
a1.channels = c1

a1.sources.r1.type = exec
a1.sources.r1.command = tail -F /opt/module/group.log
a1.sources.r1.shell = /bin/bash -c

a1.sinks.k1.type = avro
a1.sinks.k1.hostname = hadoop104
a1.sinks.k1.port = 4141

# Describe the channel
a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000
a1.channels.c1.transactionCapacity = 100

a1.sources.r1.channels = c1
a1.sinks.k1.channel = c1
```
### B数据源
配置 Source 监控端口 44444 数据流，配置 Sink 数据到下一级 Flume
```
# Name the components on this agent
a2.sources = r1
a2.sinks = k1
a2.channels = c1
# Describe/configure the source
a2.sources.r1.type = netcat
a2.sources.r1.bind = hadoop103
a2.sources.r1.port = 44444

a2.sinks.k1.type = avro
a2.sinks.k1.hostname = hadoop104
a2.sinks.k1.port = 4141

a2.channels.c1.type = memory
a2.channels.c1.capacity = 1000
a2.channels.c1.transactionCapacity = 100

a2.sources.r1.channels = c1
a2.sinks.k1.channel = c1
```

### 配置输出源
A,B 数据源聚合到一起
```
a3.sources = r1
a3.sinks = k1
a3.channels = c1

a3.sources.r1.type = avro
a3.sources.r1.bind = hadoop104
a3.sources.r1.port = 4141

a3.sinks.k1.type = logger

a3.channels.c1.type = memory
a3.channels.c1.capacity = 1000
a3.channels.c1.transactionCapacity = 100
# Bind the source and sink to the channel
a3.sources.r1.channels = c1
a3.sinks.k1.channel = c1
```

分别开启对应配置文件


`bin/flume-ng agent --conf conf/ --name
a3 --conf-file job/group3/flume3-flume-A.conf -
Dflume.root.logger=INFO,console`

`bin/flume-ng agent --conf conf/ --name
a3 --conf-file job/group3/flume3-flume-B.conf -
Dflume.root.logger=INFO,console`

`bin/flume-ng agent --conf conf/ --name
a3 --conf-file job/group3/flume3-flume-logger.conf -
Dflume.root.logger=INFO,console`

