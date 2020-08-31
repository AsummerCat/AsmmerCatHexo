---
title: ElasticSearch笔记多shard场景下相关度不准确的问题(十九)
date: 2020-08-13 10:31:43
tags: [ElasticSearch笔记]
---

## 产生原因
如果你的一个index有多个shard的话,可能搜索结果会不准确   

因为,shard中document只是一部分,默认,就在当前shard计算相关度 IDF算法(在所有document出现了多少次).  


不是全局的,可能其他shard上的相关度不一样

这样会导致搜索出来的document可能不是你想要的结果

<!--more-->
## 解决方案
```
1. 生产环境下,数据量大,尽可能实现均匀分配
数据量很大的话,其实一般情况下,在概率学的背景下,
es都是在多个shard中均匀路由数据的,路由的时候根据_id,负载均衡
   比如说有10个document,`title`都包含java,
一共有5个shard,如果负载均衡的话,其实每个shard都应该有2个doc,title都包含`java`

注意:如果数据分布均匀的话,就不会出现相关度不准确的问题


2. 测试环境下,可以将索引的primary shard设置为1个

3. 测试环境下,搜索附带search_type=dfs_query_then_fetch参数,会将local IDF取出来计算 global IDF
  
  计算一共doc的相关度分数的时候,就会将所有shard对的loacl IDF计算一下,获取出来.
在本地进行global      
  IDF分数计算,会将所有shard的doc作为上下文来行计算,也能确保准确性.
  注意: 在生产环境下不推荐这个参数,性能很差


```