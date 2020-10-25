---
title: mongodb笔记之4.0版本分布式下高可用集群(五)
date: 2020-10-26 03:38:18
tags: [mongodb笔记]
---

# 集群模式
1.M-S 主从模式 (3.0之前的版本)
2.M-A-S  类似监控主从的通信-->选举


# 集群搭建
## 配置文件
`/opt/mongodb/master-slave/slave/mongodb.cfg`
```
dbpath=/opt/mongodb/master-slave/slave/data
logpath=/opt/mongodb/master-slave/slave/logs/mongodb.log 
logappend=true
fork=true
bind_ip=0.0.0.0
port=27002 
replSet=shard002 //设置为集群 同一个副本集

```

<!--more-->