---
title: ElasticSearch笔记核心概念(一)
date: 2020-08-11 13:58:55
tags: [ElasticSearch笔记]
---

# ElasticSearch笔记基础(一)
## 优点

```
1.分布式的文档存储引擎 

2.分布式的搜索引擎和分析引擎

3.分布式:支持Pb级别的数据

提供了restful api接口
基于lucene
开箱即用
```
<!--more-->
## 核心概念
```
1. Near Realtime(NRT) 近乎实时

2.Cluster: 集群 默认集群名称(elasticsearch)

3.Node: 节点,集群中的一个节点

4.Document: 文档 ,es中的最小数据单元,一个document可以当做一条用户数据

5.Index: 索引,包含一堆有显示结构的文档数据

6.Type: 类型, 每个索引里都可以有一个或者多个Type,一个type下的document都有相同field

7.shard: 单台机子无法存储大量数据,es可以一个索引中的数据切分为多个shard,分布在多态服务器上存储

8.replica: 任何一个服务器随时可能故障或宕机,  
此时shard可能就会丢失,因此可以为每个shard创建多个replica副本.  
replica可以在shard故障时提供备用服务,保证数据可用

默认->primary shard (建立索引时一次设置,不能修改,默认5个),
replica shard(随时修改数量,默认1个->指的是每个primary shard有一个),
默认每个索引10个shard,5个primary shard,5个replica shard,

最小的高可用配置(两台服务器)
```

## 查看es的状态
```
get _cluster/health

```