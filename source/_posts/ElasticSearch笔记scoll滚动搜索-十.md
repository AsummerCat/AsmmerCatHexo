---
title: ElasticSearch笔记scoll滚动搜索(十)
date: 2020-08-11 14:08:16
tags: [ElasticSearch笔记]
---

# 作用 (scoll常用)
一次性查询10w条数据,那么性能会很差,   
此时一般采用scoll滚动查询,一批一批的差,知道所有数据都查询完处理完

<!--more-->
# 介绍
使用scoll滚动搜索,可以先搜索一批数据没然后下次再搜索一批数据,以此类推,知道搜索出全部的数据来.  

scoll搜索会在第一次搜索的时候,保存一个当时的视图快照,之后只会基于该就的视图快照提供数据搜索,如果这个期间数据变更,是不会让用户看到的.

采用基于_doc进行排序的方式,性能较高

每次发送scroll请求,我们还需要指定一个scoll参数,指定一个时间窗口,每次搜索请求只要在这个时间窗口内能完成就可以了

## 注意
scoll看起来挺像分页的,但是使用场景不一样的.  
分页主要是用来一页一页搜索,给用户看的;  
scoll主要是用来一批一批检索数据,让系统进行处理的


## 使用滑动搜索scoll提升查询效率
```
GET /index/_search?scroll=1m
{
    "query":{"match_all":{}},
    "sort":["_doc"], //排序
    "size": 1000, //单次查询数据条数
}
获取的结果会有一个scoll_id,下一次在发送scoll请求的时候,必须带上这个scoll_id


//下一次请求 只要指定个时间窗口参数就可以继续搜索了
GET 或者 POST 可以使用
URL不应该包含 index 或者 type 名字——这些都指定在了原始的 search 请求中。
scroll 参数告诉 Elasticsearch 保持搜索的上下文等待另一个 1m
scroll_id 参数
GET/_search/scroll
{
    "scroll":"1m",
    "_scroll_id":"scoll_id的内容"
}

sizze会发送给每个shard,因此每次最多会返回size * primary shard
条数据
```