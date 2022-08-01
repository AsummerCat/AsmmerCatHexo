---
title: hive的job优化
date: 2022-08-01 16:10:11
tags: [大数据,hadoop,hive]
---
# hive的job优化


## Map优化
当input文件特别复杂的时候,map执行效率慢,可以适当增加map数量
来使得每个map处理的数据量减少,从而提高任务的执行效率;



增加map的方法:根据:
computeSliteSize(Math.max(minSize,,Math.min(maxSize,blocksize)))=blocksize=128M 公式,调整maxSize最大值,让maxSize最大值低于blocksize就可以增加map的个数;

1)设置最大切片值为100字节(maxsize)
```
set mapreduce.input.fileinputformat.split.maxsize=100;
```
这样就会建立多个map进行操作;

<!--more-->

#### 小文件进行合并
1) 在map执行前合并小文件,减少map数
```
set hive.input.format=org.apache.hadoop.hive.ql.io.CombineHiveInputFormat;
```
2) 在Map-Reduce的任务结束时合并小文件的设置
   在map-only任务结束时合并小文件,默认true
```
set hive.merge.mapfiles=true;
```
在map-reduce任务结束时合并小文件,默认false
```
set hive.merge.mapredfiles=true;
```
合并文件的大小,默认256M
```
set hive.merge.size.per.task=268435456;
```
在输出文件的平均大小小于该值时,启动一个独立的map-reduce任务进行文件merge
```
set hive.merge.smallfiles.avgsize=16777216
```
#### Map端聚合
相当于map端执行combiner
```
set hive.map.aggr=true;
```
#### 推测执行
默认true;
```
set mapred.map.tasks.speculative.execution=true;
```

## Reduce端优化

#### 合理设置中Reduce数量
1) 调整reduce个数方法一
   (1)每个reduce处理的数据量默认是256MB
```
set hive.exec.reducers.bytes.per.reducer=25600000;
```
(2) 每个任务最大的reduce数,默认为1009
```
set hive.exec.reducers.max=1009;
```

2) 调整reduce个数方法二
   在hadoop的`mapred-default.xml`文件中修改
   设置每个job的reduce个数
```
set mapreduce.job.reducees=15;
```
3) reduce个数并不是越多越好
   过多的启动和初始化reduce也会消耗时间和资源;
   另外有多少个reduce就会有多少个输出文件;

#### 推测执行
默认true;
```
set mapred.reduce.tasks.speculative.execution=true;
```






