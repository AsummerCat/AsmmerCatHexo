---
title: MapReduce的Partition分区八
date: 2022-08-01 16:22:07
tags: [大数据,hadoop,MapReduce]
---
# MapReduce的Partition分区
这个概念
用于结果进行分类存储
举例:
```
要求对统计结果
->>>按照条件输出到不同文件中(分区)

比如:将统计结果按照手机归属地不同身份输出到不同文件中(分区)
```
<!--more-->
## 分区的顶级类
```
自定义分区需要继承 Partitioner<K,V>
```
## job参数,设置指定数量的ReduceTask
```
numReduceTasks

//设置指定数量的ReduceTask,会形成2个文件
job.setNumReduceTasks(2);
```
## 默认的分区 HashPartitioner
根据Key的HashCode进行分区

规则:
```
求余数:
根据Key的hashCode 和设置的ReduceTask进行取余分区

return (key.hashCode()&Integer.MAX_VALUE)%numReduceTasks;
```

## 自定义分区步骤
1.自定义类继承`Partitioner`,重写`getPartition()`方法


2.在 job中设置自定义Partitioner
```
job.setPartitionerClass(自定义分区.claas);
```

3.自定义Partitioner后,要根据自定义Partitioner的逻辑设置对应数量的ReduceTask;
```
job.setNumReduceTasks(2);
```
分区号从`0`开始

## 分区数和ReductTask任务的关系

```
如果根据规则进行分区后,分区数>ReductTask数量会抛出IO异常;

ps: (ReductTask=1的时候例外,最终输出结果就一份文件);
底层原理: 
如果分区数量>1的时候会进入定义的分区进行处理,
如果等于1,会在MapTask中调用一个内部类分区进行处理 (逻辑:分区数-1);

原因:多出的那个分区信息没有办法写入ReductTask中;
```

```

```