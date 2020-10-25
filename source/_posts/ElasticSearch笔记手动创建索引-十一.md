---
title: ElasticSearch笔记手动创建索引(十一)
date: 2020-08-11 14:09:18
tags: [elasticSearch笔记]
---

## 创建索引的语法
```
PUT /my_index1
{
    "settings":{
        "number_of_shards":3,  //primary shard数量
        "number_of_replicas": 1 //每个primary shard都有一个replica shard
    },
    "mappings":{
      "properties": {
            "my_field":{"type":"text"}
      }
       
    }
}
```
<!--more-->
## 修改索引
```
PUT /my_index/_settings
{
    "number_of_replicas":1
}

注意:
number_of_shards ->primary shard数量不能修改
_mapping设置的字段不能修改,只能新增


```

## 删除索引
```
DELETE /my_index
DELETE /index_one,idnex_two
DELETE /index_*
DELETE /_all

注意:
在 elasticsearch.yml配置文件中
action.destructive_requires_name: true
这样就不能使用`DELETE /_all`类似的操作

```