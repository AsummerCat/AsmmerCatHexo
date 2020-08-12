---
title: ElasticSearch笔记查询语句相关内容(十七)
date: 2020-08-12 21:41:31
tags: [ElasticSearch笔记]
---

# ElasticSearch笔记查询语句相关内容(十七)

## ES中match和term差别对比
match 进行分词查询

term 不对搜索的内容进行分词   直接去倒排索引中匹配  ,  
这样可能会导致倒排索引内没有这个搜索内容,
因为都分词了,  
可以尝试用字段.row来进行term的匹配  
trem 对数字,boolean,date天然支持  

<font color="red">text需要建立索引时指定为not_analyzed,才能用term query</font>
<!--more-->
```
比如:
GET /index/_search
{
    "query":{
        "constant_score":{
            "filter":{
                "term":{
                    "userID":1
                }
            }
        }
    }
}

GET /index/_search
{
    "query":{
     "match": { "userID": 1}  
    }
}

```
#### 差别
```
①`match`在匹配时会对所查找的关键词进行分词，然后按分词匹配查找

②`term`会直接对关键词进行查找 不进行分词。

一般模糊查找的时候，多用match，而精确查找时可以使用term。

```