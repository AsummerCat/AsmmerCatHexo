---
title: ElasticSearch笔记关于type的解析(十三)
date: 2020-08-11 14:10:58
tags: [ElasticSearch笔记]
---

# ElasticSearch笔记关于type的解析(十三)
# type的定义
type,是一个index中用来区分类似的数据的,
但是可能有不同的fields,而且有不同的属性来控制索引建立,分词器

field的value,在底层的lucene中建立索引的时候,全部是qpaque bytes类型,不区分类型的
<!--more-->
lucene是没有type的概念的,在document中,实际上将type作为一个document的field来存储的,即_type,es通过_type来进行type的过滤和筛选

一个index中的多个type,实际上是放在一起存储的,因此一个index下不能有多个type重名,而类型或者其他设置不同的,因为那样是无法处理的


# 关于index下的type
```
同一个index下的type是用来区分不同的document的

需要注意的是如果一个index有多个type
每个type的对应的字段都不一样
那么会产生严重的性能问题

因为index中 底层是把所有type的field字段全都揉在一起的
比如:
type1 定义了 A B C  字段
type2 定义了 D E F  字段
那么index的底层结构是:
A B C D E F

也就是说每条数据底层都至少有一半的field在底层的lucene中是空值

```

### type总结一下
```
追加时效件,将类似结构的type放在一个index下,这些type应该有多个field是相同的,
  假如说,你将两个type的field完全不同,放在同一个index下,
  那么就每条数据都至少有一半的field在底层的lucene中是空值,会有严重的性能问题

```