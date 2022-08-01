---
title: hive动态分区
date: 2022-08-01 16:06:49
tags: [大数据,hadoop,hive]
---
# hive动态分区
根据hive进行动态分区
这样可以将A表的数据导入到B表 ,根据A表的某个字段进行分区


<!--more-->
## 配置修改
在hive控制台操作

1) 开启动态分区功能(默认 true 开启)
```
set hive.exec.dynamic.partition=true;
```
2) 设置非严格模式(动态分区的模式)
```
默认 strict ,表示必须指定至少一个分区未静态分区;

nostrict模式表示允许所有分区字段都可以使用动态分区

set hive.exec.dynamic.partition.mode=nostrict;
```
3)在所有执行MR节点上,最大一共可以创建多少个动态分区
```
默认1000;

set hive.exec.max.dynamic.partitions=1000;
```
4)在每个执行的MR节点上,最大可以创建多少个动态分区
```
默认100;
根据实际情况确定,比如day 一年365天 , 分区要大于365
set hive.exec.max.dynamic.partitions.perode=100;
```

5) 整个MR job中,最大可以创建多少个HDFS文件.
```
默认为100000;

set hive.exec.max.created.fiels=100000;
```

6)当有空分区生成时,是否抛出异常
```
一般不需要设置,默认为false

set hive.error.on empty.partition=false;
```

## 动态分区的案例
将dept的数据,按照地区(ioc字段),插入到目标表dept_A中的相应分区
1.建立目标表
```
create table dept_A (id int,name string) partitioned by (ioc int) row format delimited fields terminated by '\t';
```
2.设置动态分区,并插入数据
```
set hive.exec.dynamic.partition.mode=nostrict;

insert into table dept_A partition(ioc) select deptno,dname,ioc from dept;

```
3.查看目标分区表的分区
```
show partitions dept_A;
```