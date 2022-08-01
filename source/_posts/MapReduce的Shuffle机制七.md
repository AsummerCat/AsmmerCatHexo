---
title: MapReduce的Shuffle机制七
date: 2022-08-01 16:21:34
tags: [大数据,hadoop,MapReduce]
---
# MapReduce的Shuffle机制

`Map`方法之后,`Reduce`方法之前的`数据处理过程`称之为`Shuffle`

一般叫做`混洗`的数据处理过程,
一般处理:分区->排序->合并分区->分区压缩等
<!--more-->

最终写入磁盘


## 以下内容都是在Shuffle的过程中发生的
文章记录:
```
1. MapReduce的Partition分区 

2. MapReduce的WritableComparable排序

3. MapReduce的OutputFormat输出

```
