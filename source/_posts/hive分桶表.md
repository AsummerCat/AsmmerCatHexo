---
title: hive分桶表
date: 2022-08-01 16:07:35
tags: [大数据,hadoop,hive]
---
# hive分桶表

分区表是按照数据分文件路径存储,
分桶表示按照数据文件进行分隔(1个表,会在hdfs上,生成多个文件)

## 创建分桶表
创建4个桶,根据id进行分桶
```
create table test(id int,name string) clustered by(id) into 4 buckets row format delimited fields terminated by '\t';
```
## 查看表结构
```
desc formatted test;

显示: Num Buckets:  4 
```
## 导入数据到分桶表,load的方式
```
load data inpath '/stydent.txt' into table test;
```

## 抽样查询
语句意思是: 从4个桶中提取1个桶来查询
```
select * from test tablesample(bucket 1 out 4 on id)
```