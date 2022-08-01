---
title: hive数据倾斜优化
date: 2022-08-01 16:09:29
tags: [大数据,hadoop,hive]
---
# hive数据倾斜优化

绝大部分任务都很快完成,只有一个或者少数几个任务执行的很慢甚至最终执行失败,这样的现象为数据倾斜现象;

## 单表数据倾斜优化

#### 如任务中存在`Group by`操作同时聚合函数为`count`或者`sum`可以设置参数来处理数据倾斜问题
```
1.是否开启Map端进行聚合,默认为True
2.在Map端进行聚合操作的条目数目

set hive.map.aggr=true;
set hive.groupby.managgr.checkinterval =100000;
```
有数据倾斜的时候进行负载均衡(默认是false)
```
sethive.groupby.skewindata=true
```
<!--more-->
#### 增加Reduce数量(多个Key同时导致数据倾斜)
1) 增加`reduce`个数方法
   (1) 每个Reduce处理的数量默认是256MB
```
set hive.exec.reducers.bytes.per.reducer= 256000000
```
(2) 每个任务最大的reduce数,默认为1009
```
set hive.exec.reducers.max= 1009;
```

2) 调整reduce个数方法二
   在hadoop的`mapred-default.xml`文件中修改
   设置每个job的Reduce个数
```
set mapreduce.jon.reduces =15;
```

## join数据倾斜优化
在编写join查询语句时,如果确定是由于join出现的数据倾斜,那么请做如下设置:
```
# join的键对应的记录条数超过这个值这会进行分拆,值根据具体数据量设置
set hive.skewjoin.key=100000;

# 如果是join过程出现倾斜应该设置为true

set hive.optimize.skewjoin=false;
```
如果开启了,在join过程中hive会将计数超过阀值`hive.skewjoin.key(默认100000)`的倾斜key对应额度行临时写进文件中,然后再启动另一个job做map join生成结果;
通过`hive.skewjoin.mapjoin.map.tasks=100000`参数还可以控制第二个job的mapper数量,默认1000;
```
set hive.skewjoin.mapjoin.map.tasks=100000;
```
#### 或者使用 Mapjoin