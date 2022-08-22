---
title: HBase整合Phoenix的二级索引八
date: 2022-08-22 11:35:16
tags: [大数据,HBase,Phoenix]
---

# HBase整合Phoenix的二级索引
Phoenix 二级索引,类似mysql的索引
## 二级索引配置文件
添加如下配置到 HBase 的 HRegionserver 节点的 hbase-site.xml。
```
<!-- phoenix regionserver 配置参数-->
<property>
 <name>hbase.regionserver.wal.codec</name>
 
<value>org.apache.hadoop.hbase.regionserver.wal.IndexedWALEditCod
ec</value>
</property>
```
<!--more-->

## 全局索引（global index）
默认的索引格式，创建全局索引时，会在 HBase 中建立一张新表。也就
是说索引数据和数据表是存放在不同的表中的，因此全局索引适用于多读少写的业务场景。

1. 建单个字段的全局索引 REATE INDEX
```
CREATE INDEX my_index ON my_table (my_col);
```
2. 删除索引 ROP INDEX
```
DROP INDEX my_index ON my_table
```

3. 查看执行计划 explain
```
explain select id,name,addr from student1 where age = 10;
```
## 包含索引（covered index）
创建携带其他字段的全局索引（本质还是全局索引）。
1.创建包含索引
```
CREATE INDEX my_index ON my_table (v1) INCLUDE (v2);
```

## 本地索引（local index）
Local Index 适用于写操作频繁的场景。
索引数据和数据表的数据是存放在同一张表中（且是同一个 Region），避免了在写操作
的时候往不同服务器的索引表中写索引带来的额外开销。
my_column 可以是多个

1. 创建本地索引
```
CREATE LOCAL INDEX my_index ON student1 (age,addr);
```