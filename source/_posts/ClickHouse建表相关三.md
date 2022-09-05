---
title: ClickHouse建表相关三
date: 2022-09-05 17:20:21
tags: [大数据,ClickHouse]
---
# ClickHouse建表相关

## 语句
<font color='red'>注意: 主键是允许重复的,
order by 中的字段中必须有
要求：主键必须是 order by 字段的前缀字段。
比如 order by 字段是 (id,sku_id) 那么主键必须是 id 或者(id,sku_id)
</font>
### 1）建表语句
```
create table t_order_mt(
 id UInt32,
 sku_id String,
 total_amount Decimal(16,2),
 create_time Datetime
) engine =MergeTree
 partition by toYYYYMMDD(create_time)
 primary key (id)
 order by (id,sku_id);
```
<!--more-->
### 2）插入数据
```
insert into t_order_mt values
(101,'sku_001',1000.00,'2020-06-01 12:00:00') ,
(102,'sku_002',2000.00,'2020-06-01 11:00:00'),
(102,'sku_004',2500.00,'2020-06-01 12:00:00'),
(102,'sku_002',2000.00,'2020-06-01 13:00:00'),
(102,'sku_002',12000.00,'2020-06-01 13:00:00'),
(102,'sku_002',600.00,'2020-06-02 12:00:00');
```
MergeTree 其实还有很多参数(绝大多数用默认值即可)，但是三个参数是更加重要的，
也涉及了关于 MergeTree 的很多概念。


## primary key 主键(可选)
```
ClickHouse 中的主键，和其他数据库不太一样，它只提供了数据的一级索引，但是却不
是唯一约束。这就意味着是可以存在相同 primary key 的数据的。
主键的设定主要依据是查询语句中的 where 条件。
```
## order by（必选）
```
order by 设定了分区内的数据按照哪些字段顺序进行有序保存。
order by 是 MergeTree 中唯一一个必填项，甚至比 primary key 还重要，因为当用户不
设置主键的情况，很多处理会依照 order by 的字段进行处理（比如后面会讲的去重和汇总）。
要求：主键必须是 order by 字段的前缀字段。
比如 order by 字段是 (id,sku_id) 那么主键必须是 id 或者(id,sku_id)
```

## partition by 分区(可选)
```
如果不填 只会使用一个分区
```

## 二级索引
语法:
```
INDEX a total_amount TYPE minmax GRANULARITY 5

其中 GRANULARITY N 是设定二级索引对于一级索引粒度的粒度。
```
例子:
```
create table t_order_mt2(
 id UInt32,
 sku_id String,
 total_amount Decimal(16,2),
 create_time Datetime,
INDEX a total_amount TYPE minmax GRANULARITY 5
) engine =MergeTree
 partition by toYYYYMMDD(create_time)
  primary key (id)
 order by (id, sku_id);
```
