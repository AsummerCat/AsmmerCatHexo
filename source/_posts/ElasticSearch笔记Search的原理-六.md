---
title: ElasticSearch笔记Search的原理(六)
date: 2020-08-11 14:05:51
tags: [elasticSearch笔记]
---


## 搜索超时(Search TimeOut)
```
默认情况下 是没有做搜索超时的

超时时间指的是单个shard的超时,也就是搜索:如果是多个shard,每个shard的查询时间要小于超时时间

手动添加参数:

GET /_search?timeout=30ms

```
<!--more-->

# 搜索原理
```
GET 
/_search                            所有索引,所有type下的所有数据都搜索出来
/index1/_search                     指定一个index,搜索其下所有type的数据
/index1,index2/_search              同时搜索两个index下的所有数据
/*1,*2/_search                      根据通配符去匹配多个index的所有数据
/index1/type1/_search               搜索index下指定type下的所有数据
/index1/type1,type2/_search         搜索index下多个type下的所有数据
/index1,index2/type1,type2/_search  搜索多个index下多个type下的所有数据
/_all/type1,type2/_search           _all 标识搜索所有index下的指定type

```
#### 请求流程
```
client发送一个搜索请求,会把请求达到所有的primary shard上去执行.
因为每个shard都包含部分数据,所有每个shard上都可能会包含搜索请求的结果

 但是如果primary shard有replica shard 那么请求也可以打到replica shard上去
```


#### 分页搜索
```
size:查询几条数据    (分页)
from:从第几条数据开始查
to: 到第几条数据为止


GET /_search?size=10     查询10条记录

GET /_search?size=10&from=0   从0开始查询10条

GET /_search?size=10&from=20  从20开始查询10条
```

#### deep paging性能问题
##### 什么是deep paging问题
深度搜索
```
深度搜索的问题

简单来说:
我们只要10条数据
就是搜索的特别深,比如总共有10000条数据,每页是10条数据,这个时候,你要搜索到最后一页

每个shard都返回相同的数据每个shard返回1w条数据  协调节点进行相关度排序  选出10条你相关度最高的结果返回给客户端

搜索过程的时候,就需要在协调节点上保存大量的数据,还需要进行大量的排序,
耗费CPU耗费内存,所以deep paging的问题 我们要避免

```