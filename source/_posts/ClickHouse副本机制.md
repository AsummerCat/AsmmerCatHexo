---
title: ClickHouse副本机制
date: 2022-09-05 17:20:59
tags: [大数据,ClickHouse]
---
# ClickHouse副本机制

# 副本机制
没有主从的区分
互为副本,都可以写入,利用zk进行同步

注意:
主要在建表的时候确定副本引擎
由于`ClickHouse`系统上有支持
<!--more-->
```
ReplicatedMergeTree
ReplicatedSummingMergeTree
ReplicatedReplacingMergeTree
ReplicatedAggregatingMergeTree
ReplicatedCollapsingMergeTree
ReplicatedVersionedCollapsingMergeTree
ReplicatedGraphiteMergeTree
```
这些基本都在原先的表结构模型中只是加了`Replicated`

## 例子:
```
CREATE TABLE table_name
(
    EventDate DateTime,
    CounterID UInt32,
    UserID UInt32,
    ver UInt16
) ENGINE = ReplicatedReplacingMergeTree('/clickhouse/tables/{layer}-{shard}/table_name', '{replica}')
PARTITION BY toYYYYMM(EventDate)
ORDER BY (CounterID, EventDate, intHash32(UserID))
SAMPLE BY intHash32(UserID);
```

## 参数详解
```
ReplicatedReplacingMergeTree('/clickhouse/table/表名/分片/table_name', '副本名称')

ReplicatedReplacingMergeTree('/clickhouse/table/01/tearcher', 'tearcher_one')

第一个参数: zk路径 
第二个参数: 副本名称 需要唯一
```