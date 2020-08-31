---
title: ElasticSearch笔记使用function_score提高自定义计算分数(二十一)
date: 2020-08-23 15:40:18
tags: [ElasticSearch笔记]
---

假设场景:   
我们需要
按照看帖人数的多少进行排序帖子的顺序

`利用`function_score实现
<!--more-->
## 实现
```
GET /index/_search
{
    "query":{
        "function_score":{
            "query":{
                "multi_match":{
                    "query":"java spark",
                    "fields":["title","content"]
                }
            },
            "field_value_factor":{
                "field":"follower_num",
                "modifier":"log1p",
                "factor": 0.1
            },
            "boost_mode":"sum",
            "max_boost": 1.5
        }
    }
}

每个doc的分数乘以follow_num 就是最终的分数 

log1p能保证计算的分数不为0,

factor :该值(follower_num)的权重大小

boost_mode, 可以决定分数与指定字段的值如何计算 ,multiply,sum,min,max,replace

max_boost ,限制计算出来的分数不要超过max_boost指定的值
```