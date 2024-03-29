---
title: ElasticSearch笔记利用scoll重建错误的index(十五)
date: 2020-08-11 14:11:54
tags: [elasticSearch笔记]
---

# 基于scoll滚动搜索和alias别名实现零停机 重建索引
## 发生场景
```
比如插入 2017-01-01 会自动转为date类型
如果后期插入字符串 会报错,
并且无法修改mapping的数据类型
```
<!--more-->
## 解决方案
唯一方法:  
重新建立一个索引,将旧索引的数据查询出来,再导入新索引

#### 第1️⃣步 重建索引

旧索引

```

PUT /old_index/_doc/4
{
    "age": 24,
    "date":"2017-01-06"
}

----------------------------
  索引的mapping
{
  "old_index" : {
    "mappings" : {
      "properties" : {
        "age" : {
          "type" : "long"
        },
        "date" : {
          "type" : "date"
        }
      }
    }
  }
}

```



创建一个正确类型的index

```
//创建索引
PUT /new_index?pretty

PUT /new_index/_mappings
{
  "properties":{
    "date":{
      "type": "text",
      "analyzer": "standard"
    }
   
	}
}
```



#### 第2️⃣步 使用scroll查询出旧索引的数据
一个field的设置是不能被修改的,如果要是该一个field,name应该重新按照新的的mapping,建立一个index,然后将数据批量查询出来,重新用用bulk api 写入index中.

批量查询的时候,建议采用scroll api,并且采用多线程并发的方式来reindex数据,每次scoll就查询指定日期的一段数据,交给一线程即可
```
GET /old_index/_search?scroll=1m
{
    "query":{
        "range":{
            "date":{
                "gte":"2017-01-01",
                "lt":"2017-02-01"
            }
        }
    },
    "sort":["_doc"],
    "size": 1000
}

或者:
查询所有数据 一批一批来
GET /old_index/_search?scroll=1m
{
    "query":{
        "match_all":{}
    },
    "sort":["_doc"],
    "size": 1000
}
```
#### 第3️⃣步 使用bulk api将scoll查出来的一批数据写入新索引
```
POST /_bulk
{"index":{"_index":"new_index","_id":"2"}}
{"title":"2017-01-01"}

写入到新索引中

```
反复 2️⃣3️⃣步批量写入新索引


#### 第4️⃣步 基于alias对client透明切换index
创建一个别名`my_index` 先给外部使用
```
PUT /old_index/_alias/my_index_test

client对old_index进行操作
reindex操作,完成之后,切换v1到v2
```
#### 第5️⃣步 删除别名指向的旧index,加入新的index
```
POST /_aliases
{
    "actions":[
    {"remove":{"index":"old_index","alias":"my_index_test"}},
    {"add":   {"index":"new_index","alias":"my_index_test"}}
    ]
}
```

#### 第6️⃣步 测试数据

```
别名指向了 new_index

GET /my_index_test/_mapping
GET /my_index_test/_search
{
  "query": {
    "match_all": {
    }
}}

测试数据->都是指向new_index
```

