---
title: hive任务整体优化
date: 2022-08-01 16:10:47
tags: [大数据,hadoop,hive]
---
# hive任务整体优化

## Fetch抓取
fetch抓取是指:hive中对于某些情况的查询可以不必使用`MapReduce`计算,可以简单读取对应的存储目录下的文件,然后输出查询结果到控制台;

比如`select * from emp;`


修改`hive-default.xml.template`文件中的`hive.fetch.task.conversion`默认值为more,老版本hive默认是`minimal`,
将值修改为`more`后,在全局查找,字段查找,limit查找等都不走`MapReduce`

<!--more-->

## 本地模式
大数据job是需要`hadoop`提供完整可扩展性来处理大数据集的,在数据量小的时候,可以使用本地模式来在单机上处理所有任务,
对于小数据集,执行时间可以明显被缩短

设置参数:
```
//开启本地模式
set hive.exec.mode.local.auto=true;

//设置本地模式的最大输入数据量,当输入数据量小于这个值时采用local模式,默认134217728,即128M
set hive.exec.mode.local.auto.inputbytes.max=50000000;

//设置本地模式最大输入文件个数,当输入文件个数小于这个值时采用local模式,默认为4;
set hive.exec.mode.local.auto.input.files.max=10;

````

## 并行执行
让hive查询的多个阶段,不依赖的部分进行并行执行;
如果job中并行阶段增多,那么集群利用率就会增加;
```
//打开任务并行执行,默认为fasle;
set hive.exec.parallel=true;

//同一个sql允许最大并行度,默认为8
set hive.exec.parallel.thread.number=16;

```

## 严格模式
hive可以通过设置防止一些危险操作:
1)分区表不使用分区过滤
```
set hive.strict.checks.no.partition.filter=true
```
除非where语句中包含分区字段过滤条件来现在范围,否则不允许执行;

2) order by 没有limit过滤
```
set hive.strict.checks.orderby.no.limit=true;
```
对于使用了order by语句的查询,要求必须使用limit语句;

3) 笛卡尔积
```
set hive.strict.checks.cartesian.product=true;
```
限制笛卡尔积的查询;


