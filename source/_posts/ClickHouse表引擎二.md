---
title: ClickHouse表引擎二
date: 2022-09-05 17:19:53
tags: [大数据,ClickHouse]
---
# ClickHouse表引擎

## 表引擎的使用
表引擎是 ClickHouse 的一大特色。可以说， 表引擎决定了如何存储表的数据。包括：
➢ 数据的存储方式和位置，写到哪里以及从哪里读取数据。 ➢ 支持哪些查询以及如何支持。
➢ 并发数据访问。
➢ 索引的使用（如果存在）。
➢ 是否可以执行多线程请求。
➢ 数据复制参数。
表引擎的使用方式就是必须显式在创建表时定义该表使用的引擎，以及引擎使用的相关参数。

<font color='red'>特别注意：引擎的名称大小写敏感</font>

<!--more-->

# 表引擎类型

## TinyLog
以列文件的形式保存在磁盘上，不支持索引，没有并发控制。一般保存少量数据的小表，
生产环境上作用有限。可以用于平时练习测试用。
如：
```
create table t_tinylog ( id String, name String)engine=TinyLog;
```

## Memory
内存引擎，数据以未压缩的原始形式直接保存在内存当中，服务器重启数据就会消失。
读写操作不会相互阻塞，不支持索引。简单查询下有非常非常高的性能表现（超过 10G/s）。
一般用到它的地方不多，除了用来测试，就是在需要非常高的性能，同时数据量又不太大（上限大概 1 亿行）的场景

## MergeTree
ClickHouse 中最强大的表引擎当属MergeTree（合并树）引擎及该系列（*MergeTree）
中的其他引擎，支持索引和分区，地位可以相当于 innodb 之于 Mysql。而且基于MergeTree，还衍生除了很多小弟，也是非常有特色的引擎。



## ReplacingMergeTree (MergeTree变种)
提供最终一致性去重的功能
```
create table t_order_rmt(
 id UInt32,
 sku_id String,
 total_amount Decimal(16,2) ,
 create_time Datetime 
) engine =ReplacingMergeTree(create_time)
 partition by toYYYYMMDD(create_time)
 primary key (id)
 order by (id, sku_id);
```
`ReplacingMergeTree()`填入的参数为版本字段，重复数据保留版本字段值最大的。
如果不填版本字段，默认按照插入顺序保留最后一条。

注意: 这个只会在合并的过程中去重;

## SummingMergeTree(MergeTree变种)
以 `SummingMergeTree（）`中指定的列作为汇总数据列

对于不查询明细，只关心以维度进行汇总聚合结果的场景。如果只使用普通的MergeTree
的话，无论是存储空间的开销，还是查询时临时聚合的开销都比较大。
ClickHouse 为了这种场景，提供了一种能够“预聚合”的引擎 SummingMergeTree
```
create table t_order_smt(
 id UInt32,
 sku_id String,
 total_amount Decimal(16,2) ,
 create_time Datetime 
) engine =SummingMergeTree(total_amount)
 partition by toYYYYMMDD(create_time)
 primary key (id)
 order by (id,sku_id );
```

# MateriailzeMySQL引擎
跟`canal`,`Maxwell` 差不多都是基于binlog日志,写入数据

`20.8.2.3`版本之后加入的特性
