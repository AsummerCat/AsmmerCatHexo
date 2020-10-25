---
title: ElasticSearch笔记相关度评分(八)
date: 2020-08-11 14:07:15
tags: [elasticSearch笔记]
---

## TF&TDF算法
简单来说就是计算出,  
一个索引中的文本,与搜索文本,他们之间的关联匹配程度


## 算法内容
es使用的是  
`Term frequency`与`Inverse document frequency`算法  

---

```
1.Term frequency:
搜索文本中的各个词条在field文本中出现了多少次,出现次数越多,就越相关

2.Inverse document frequency:
搜索文本中的各个词条在整哥索引的所有文档出现了多少次,出现的次数越多,就越不相关

这里的意思是:
hello world
比如hello 出现了3000次,
world 出现了10000次;
这样表明 world的相关度越低 因为无关性可能大幅度提升了


3.Field-length norm:
field长度,field越长,相关度越低
```

<!--more-->
## 评分如何计算出来的
```
就是根据
`Term frequency`与`Inverse document frequency`算法  
进行计算的

可以使用  ?explain 来查看相关内容

```