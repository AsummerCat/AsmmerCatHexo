---
title: ElasticSearch笔记基于nested_Object类型实现关系型数据建模(二十八)
date: 2020-08-31 21:58:06
tags: [elasticSearch笔记]
---

## 前言
如果不是使用`nested object`类型创建关系数据建模的话,
可能会导致数据搜索不正确

```
比如:
 对象里面: 有个list
 [{name="小明","age"=16},{name="小东","age"=18}]
 普通的object类型 会将他拆词为
  name [小明 ,小东]
  age [16,18]
  这样会导致我们使用搜索的时候会出现结果不太一致的情况
  会变成数组 进行倒排索引
  
  
  如果使用 nested object类型的话
  数据结构就还是关系型的那一套结构
 
```

<!--more-->

##  创建语法
```
PUT  /website
{
  "mappings": {
     "properties":{
        "comments":{
          "type":"nested",
          "properties": {
            "name":{"type":"text"},
            "content":{"type":"text"},
            "age":{"type": "long"}
          }
        }
    }
  }
}
```
## 查询语句
```
GET /website/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "nested": {
            "path": "comments",
            "score_mode": "max", 
            "query": {
              "bool": {
                "must": [
                  {"match": {
                    "comments.name": "小明"
                  }},
                  {
                    "match": {
                      "comments.content": "下"
                    }
                  }
                ]
              }
            }
          }
          
        }
      ]
    }
  }
}

'score_mode' : max, min, avg, none
这个表示: 合并总分的策略
```


---



# 测试语句
## 原object类型
```

PUT /website?pretty

PUT /website/_doc/1
{
  "title":"夏天快来了",
  "comments":[
   {
      "name":"小东",
      "age":18
   },
   {
      "name":"小明",
      "age":19
   }
    ]
}

PUT /website/_doc/2
{
  "title":"冬天快来了",
  "comments":[
   {
      "name":"小东",
      "age":20
   },
   {
      "name":"黄药师",
      "age":21
   }
    ]
}



GET /website/_search
{
  "query": {
    "bool": {
      "must": [
        {"match": {
          "comments.name": "东"
        }},
         {"match": {
          "comments.age": 21
        }}
        
      ]
    }
  }
}



```
输出结果:  
这里的结果虽然能输出,但是不是完全匹配的数据  
`age=21`这个应该匹配不到小东的 所以有误差
```
{
  "took" : 3,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 1.3411673,
    "hits" : [
      {
        "_index" : "website",
        "_type" : "_doc",
        "_id" : "2",
        "_score" : 1.3411673,
        "_source" : {
          "title" : "冬天快来了",
          "comments" : [
            {
              "name" : "小东",
              "age" : 20
            },
            {
              "name" : "黄药师",
              "age" : 21
            }
          ]
        }
      }
    ]
  }
}

```

# 修改后的nested object类型
```
GET /website/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "title": "天"
          }
        },
        {
          "nested": {
            "path": "comments",
            "query": {
              "bool": {
                "must": [
                  {
                    "match": {
                      "comments.name": "东"
                    }
                  },
                  {
                    "term": {
                      "comments.age": 21
                    }
                  }
                ]
              }
            }
          }
        }
      ]
    }
  }
}

```
查询结果:
不匹配`comments`里面的完整的一条记录
所以出不来数据
'age'是21  
'name'是小东 构成不了一条记录

```
{
  "took" : 1,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 0,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [ ]
  }
}

```

基于nested object的建模,有个不好的地方,就是会冗余数据,维护成本高.



# 父子关系数据模型建模 (提高netsted的性能)
父子关系数据模型,相对于nested数据模型来说,优点是父doc和子doc互相之间不会影响  
父子关系元数据映射,用于确保查询时候的高性能,但是有一个限制,就是父子数据必须存在一个`shard`上  

多个实体都分割开来,每个实体之间都通过一些关联方式进行了父子关系的建模

## 建模语法
```
PUT /cpmpany
{
    "mappings":{
        "rd_center":{},
        "employee":{
            "_parent":{
                "type":"rd_center"
            }
        }
    }
}

父子咕逆袭 _parent来指定父type

```
## 建立关联
`parent=id`用来建立关系
```
PUT /company/employee/1?parent=1
{
    "name":"张三",
    "birthday":"1970-10-24",
    "hobby":"爬山"
}
```
此时,parent-child关系,就确保了说,父doc和子doc都是保存在一个shard上.  
内部原理还是doc  routing .`employee`和`rd_center`的数据,都会用parent;

## 插入语法
```
POST /company/employee/_bulk
{"index":{"_id":2,"parent:"1"}}
{"name":"李四","birthday":"1982-05-16","hobby":"游泳"}
{"index":{"_id":3,"parent:"2"}}
{"name":"张三","birthday":"1970-08-20","hobby":"跳舞"}
{"index":{"_id":4,"parent:"3"}}
{"name":"王五","birthday":"1990-03-19","hobby":"旅游"}


```
## 父子建模互相搜索的语法
### 根据子节点搜索 父节点数据
`has_child`
```
GET /company/rd_center/_search
{
    "query":{
        "has_child":{
            "type":"employee",
            "score_mode":"max", //匹配程度最高的
            "query":{
                "range":{
                    "birthday":{
                        "gte":"1980-01-01"
                    }
                }
            }
        }
    }
}

```
### 根据父节点搜索 子节点数据
`has_parent`
```
GET /company/employee/_search
{
    "query":{
        "has_parent":{
            "type":"rd_center",
            "query":{
                "match":{
                    "country.keyword":"中国"
                }
            }
        }
    }
}

```

### 搜索子节点有两个以上的父节点数据
`min_children` 最少有几个子节点的数据
```
GET /company/rd_center/_search
{
    "query":{
        "has_child":{
            "type":"employee",
            "min_children":2, //最少有两个子节点的数据的父节点
            "query":{
                "match_all":{}
            }
        }
    }
}

```

### 祖孙三层的节点数据如何加入
`routing`参数的详解,必须跟`grandparent`相同,否则会有问题

如果仅仅指定一个parent,那么用的是rd_center的id去路由,这就导致祖孙三层数据不会在一个shard上
```
PUT /company/employee/1?parent=1&routing=1
{
    "name":"张三",
    "dob":"1970-10-24",
    "hobby"爬山"
}

```
### 祖孙三层的查询
```
GET /company/country/_search
{
    "query":{
        "has_child":{
            "type":"rd_center",
            "query":{
                "has_child":{
                    "type":"employee",
                    "query":{
                        "hobby":"爬山"
                    }
                }
            }
        }
    }
}


```