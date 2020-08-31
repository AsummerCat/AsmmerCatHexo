---
title: ElasticSearch笔记实现搜索提示(三十一)
date: 2020-08-31 22:00:11
tags: [ElasticSearch笔记]
---

## 介绍
`suggest completion suggest`, 自动完成,  搜索推荐,搜索提示-->自动完成 auto completion

<!--more-->

## 语法
### 首先需要创建一个completion的字段类型
```
PUT /my_index
{
    "mappings":{
        "properties":{
            "title":{
                "type":"text",
                "analyzer":"ik_max_word",
                "fields":{
                    "suggest":{ //字段
                        "type":"completion",
                        "analyzer":"ik_max_word"
                    }
                }
            }
        }
    }
}

```
`completion` ,es实现的时候,是非常高性能的,会构建不是倒排索引,也不是正排索引.  
就是专门用于进行前缀搜索的一种特殊的数据结构,而且会全部放在内存中,所以`auto completion`进行的前缀搜索提示,性能是非常高的


### 搜索
```
GET /my_index/_search
{
    "suggest":{ //语法开头
        "my-suggest":{
            "prefix":"大话西游",
            "completion":{
                "field":"title.suggest"
            }
        }
    }
}

能关联搜索

'prefix': 表示搜索关键词前缀


```