---
title: ElasticSearch笔记filter执行原理深度解析(十八)
date: 2020-08-12 21:42:24
tags: [elasticSearch笔记]
---

## bitset机制和caching机制


# 流程大纲
```
1.在倒排索引中查找搜索串,获取document list

2.为每个在倒排索引中搜索到的结果,构建一个bitset,[0,0,0,1,0,1]  
这个就是相当于存在某个document中就标记为1

3.遍历每个过滤条件对应的bitset(_doc),
优先从最稀疏的开始搜索(这样命中快->类似小表驱动大表),查找满足所有条件的document

4.(缓存满足过滤条件的_doc)caching bitset,跟踪query,在直接256个query中超过一定次数的过滤条件, 
缓存其bitset.对于小segment(<1000 或<3%),不缓存bitset. 
这样的作用: 下次还有这个条件过来的时候,就不用查询扫描倒排索引,反复生成bitset,可以大幅度提升性能

```
<!--more-->
```
以下是注意部分:  

5.filter大部分情况下来说,在query之前执行,先尽量过滤尽可能多的数据
因为:
query:会计算doc对搜索条件的相关度分数,还会根据分数排序
filter:只是简单过滤出想要的数据,不计算相关度,不排序

6.如果document有新增或者修改,那么cached bitset会被自动更新


7.以后只要是有相同的filter条件的,会直接来使用这个过滤条件对应的cached bitset  

可以直接去缓存拿,加快查询速度
```