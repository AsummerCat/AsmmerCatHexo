---
title: MapReduce基础一
date: 2022-08-01 16:17:53
tags: [大数据,hadoop,MapReduce]
---
# MapReduce进程
一个完整额度MapReduce程序在分布式运算时有三类实例进程

```
1.MrAppMaster: 负责整个程序的过程调度和状态协调

2.MapTask: 负责Map阶段的整个数据处理流程

3.ReduceTask: 负责Reduce阶段的整个数据处理流程

MapTask ReduceTask 可以算是 yarn child的一个子进程
```

<!--more-->

# 常用数据序列化类型

| java类型 | Hdoop Writable类型 |
| --- | --- |
|  Boolean  |BooleanWritable|
| Byte | ByteWritable |
|  Int| IntWritable|
| Float | FloatWritable|
| Long | LongWritable|
| Double | DoubleWritable|
| String | Text|
| Map | MapWritable|
| Array | ArrayWritable|
| Null | NullWritable|


# 编码规范
三个类
```
1. 编写一个 继承Mapper<K,V>类

2. 编写一个 继承 Reducer类 

3. 编写调用类 job任务调度主方法

```

## Mapper类

```
extends Mapper<Object ,Text,Text,IntWritable>


extends Mapper<输入key的类型 ,输入value的类型,输出key的类型,输出value的类型>

需要实现内部的map方法 

```

## Reducer类
```
extends Reducer<Object ,Text,Text,IntWritable>


extends Reducer<输入key的类型 ,输入value的类型,输出key的类型,输出value的类型>

需要实现内部的reduce方法  

```
Reducer的输入类型 对应Mapper的输出类型
