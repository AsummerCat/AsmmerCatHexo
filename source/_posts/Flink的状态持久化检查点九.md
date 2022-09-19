---
title: Flink的状态持久化检查点九
date: 2022-09-19 17:00:42
tags: [大数据,Flink]
---
# Flink的状态持久化检查点

## 检查点
要是用检查点 需要在代码中手动开启
```
val env = StreamExecutionEnvironment.getEnvironment
//进行检查点的保存 10秒保存一次
env.enableCheckpointing(10000)
```

需要实现`CheckpointedFunction` 这个接口

<!--more-->
## “保存点”（savepoint）的功能
保存点在原理和形式上跟检查点完全一样，也是状态持久化保存的一个快照；区别在于，保存点是自定义的镜像保存，所以不会由 Flink自动创建，而需要用户手动触发。这在有计划地停止、重启应用时非常有用。

## 状态后端（State Backends）
检查点的保存
（1）哈希表状态后端（HashMapStateBackend）
保存在JVM堆中
```

```
（2）内嵌 RocksDB状态后端（EmbeddedRocksDBStateBackend）
持久化保存在磁盘上
默认存储在TaskManager的本地数据目录中
```

```

### 设置方式
（1）配置默认的状态后端
在 flink-conf.yaml 中，可以使用 state.backend来配置默认状态后端。
配置项的可能值为 hashmap，这样配置的就是 HashMapStateBackend；也可以是 rocksdb，这样配置的就是 EmbeddedRocksDBStateBackend。下面是一个配置 HashMapStateBackend 的例子：
```
# 默认状态后端
state.backend: hashmap
# 存放检查点的文件路径
state.checkpoints.dir: hdfs://namenode:40010/flink/checkpoints
```

（2）为每个作业（Per-job）单独配置状态后端
每个作业独立的状态后端，可以在代码中，基于作业的执行环境直接设置。代码如下：
```
val env = StreamExecutionEnvironment.getExecutionEnvironment();
env.setStateBackend(new HashMapStateBackend())

val env = StreamExecutionEnvironment.getExecutionEnvironment();
env.setStateBackend(new EmbeddedRocksDBStateBackend())

```
需要注意，如果想在 IDE 中使用 EmbeddedRocksDBStateBackend，需要为 Flink 项目添加依赖：
```
<dependency>
 <groupId>org.apache.flink</groupId>
 <artifactId>flink-statebackend-rocksdb_2.11</artifactId>
 <version>1.13.0</version>
</dependency>
```