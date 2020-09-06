---
title: ElasticSearch笔记运维集群jvm和服务器内存分配的最佳实践(三十九)
date: 2020-09-06 22:07:05
tags: [ElasticSearch笔记]
---

#  es的jvm内存大小设置
大于大部分生产环境来说的 2g的jvm heap size 太小了

1.可以设置个环境变量来替换 es的默认配置
2.直接修改配置文件 `jvm.options`
3.启动设置参数 

<!--more--> 

## 内存分配上的划分
1. jvm heap size 最好50%  
2. os cache也是50%
3. 最好不要给jvm分配超过32G的内存 默认是开启CMS的

如果你不需要的`fielddata`,也就是我们不要对任何分词的string field进行聚合操作的时候,因为`fielddata`是要用`jvm heap size`的,那我们可以给`jvm heap size`分配更少的内存,这样es的新呢个反而会更好. 

es的性能很大一部分,其实是由多少内存留给操作系统的os  
1.cache ->倒排索引(为了全文检索)  
2.jvm heap->正排索引(为了聚合查询)

因为更多的内存留给`lucene`用`os cache`提升索引读取性能,同事es的`jvm heap`的耗时会更少


一般来说-> 如果你的es要处理的数据量上亿的话,建议,就是用64的内存的机器比较合适,有5台左右差不多了,给jvm options 32g ,os cache 32g


## 在32G内存以内具体设置heap为多大?
最好设置在31G以内 保留1G以上给其他文件