---
title: hive的HQL语句优化
date: 2022-08-01 16:08:44
tags: [大数据,hadoop,hive]
---
# hive的HQL语句优化

## 可能存在数据倾斜的情况
需要开启Map端聚合参数配置
1.是否开启Map端进行聚合,默认为True
```
set hive.map.aggr=true;
```
2.在Map端进行聚合操作的条目数目
```
set hive.groupby.managgr.checkinterval =100000;
```
3.有数据倾斜的时候进行负载均衡(默认是false)
```
sethive.groupby.skewindata=true
```
<!--more-->

当选项设定为true,生成的查询计划会有两个MR job;

第一个MRjob中,Map的输出结果会随机分布到Reduce中,每个Reduce做部分聚合操作,并输出结果,这样处理的结果<font color='red'>是相同的Group By Key 有可能被分发到不同的Reduce</font>中,从而达到负载均衡的目的;

第二个MRjob再根据预处理的数据结果按照Group By Key分布到Reduce中(这个过程,可以保证相同的Group By Key被分配到同一个Reduce中),最后完成最终的聚合操作(虽然能解决数据倾斜,但是不能让运行速度的更快);

## join语法优化
改造:`left semi join`语法
 ```
 
 select a from test_a join b表 on a.id=b.id
 
 改为:
  select a from test_a left semi join b表 on a.id=b.id
 ```

## CBO优化
底层进行查询优化
```
查询前设置以下参数:

set hive.cbo.enable=true;
set hive.compute.query.using.stats=true;
set hive.stats.fetch.column.stats=true;
set hive.stats.fetch.partition.stats=true;

```

## 谓语下推
wehere条件提前执行处理,减少查询数据量
```
默认是true
set hive.optimize.ppd=true;
```

## MapJoin  (大表和小表)
将join双方比较小的表直接分发给各个Map进程的内存中,在Map中进行join操作,这样就不用进行Reduce步骤;

如果不指定Mapjoin或者不符合MapJoin的条件,会在Reduce阶段完成join.容易发生数据倾斜;

1) 开启Mapjoin参数
```
默认开启
set hive.auto.convert.join=true; 

大表小表的阀值设置(默认25M以下认为是小表)
set hive.mapjoin.smalltable.filesize=25000000;
```
这样的话 要用大表驱动小表,
如果用小表驱动(left join)大表 MapJoin会失效;

## 大表,大表的 SMB join
SMB join : Sort Merge Bucket Join
这个使用分桶来进行join

两个大表使用 id 进行分桶, 那么对应id的数据会在对应的桶里;

设置参数: 开启桶join
```
set hive.optimize.bucketmapjoin=true;
set hive.optimize.bucketmapjoin.sortedmerge=true;
set hive.input.format=org.apache.hadoop.hive.ql.io.BucketizedHiveInputFormat;
```
测试:
```
inset overwrite table jointable
select b.id,b.t 
from a_buck1 s
join b_buck2 b
on s.id=b.id;
```

