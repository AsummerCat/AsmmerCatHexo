---
title: Flink的分区四
date: 2022-09-19 16:56:38
tags: [大数据,Flink]
---
# Flink的分区

## 物理分区（Physical Partitioning）

#### 1. 随机分区（shuffle）
最简单的重分区方式就是直接“洗牌”。通过调用 DataStream 的shuffle()方法，将数据随机地分配到下游算子的并行任务中去。
随机分区服从均匀分布（uniformdistribution），所以可以把流中的数据随机打乱，均匀地传递到下游任务分区

```
// 读取数据源，并行度为 1
 val stream = env.addSource(new ClickSource)
 // 经洗牌后打印输出，并行度为 4
 stream.shuffle.print("shuffle").setParallelism(4)
```
<!--more-->
#### 2.轮询分区（Round-Robin）
按照先后顺序将数据做依次分发
```
// 读取数据源，并行度为 1
 val stream = env.addSource(new ClickSource)
 // 经轮询重分区后打印输出，并行度为 4
 stream.rebalance.print("rebalance").setParallelism(4)
```

#### 3.广播（broadcast）
将输入数据复制并发送到下游算子的所有并行任务中去。
```
// 读取数据源，并行度为 1
 val stream = env.addSource(new ClickSource)
 // 经广播后打印输出，并行度为 4
 stream.broadcast.print("broadcast").setParallelism(4)
```