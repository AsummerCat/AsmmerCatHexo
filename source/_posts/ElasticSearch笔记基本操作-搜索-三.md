---
title: ElasticSearch笔记基本操作+搜索(三)
date: 2020-08-11 14:02:26
tags: [elasticSearch笔记]
---

# 操作
## 基本操作

#### 插入文档
```
//指定id
PUT /test_index/_doc/1
{
  "name":"小明",
  "age":18
}

//使用随机id
POST /test_index/_doc
{
  "name":"小明",
  "age":18
}
```
#### 更新文档 (全量替换)

语句跟插入相同

<!--more-->

```
全量替换
PUT /test_index/_doc/1
{
  "name":"小明更新了",
  "age":18
}

PUT /test_index/_doc/1
更新文档指定字段:
PUT /test_index/_doc/1/_update

//基于最新版本号重试5次:
POST /test_index/_update/1?retry_on_conflict=5
```
#### 更新文档 (部分字段更新)

更新文档指定字段:
```
POST /test_index/_update/1
{
    "doc": {
    	"age":19
    }
}
```
#### 更新文档 (指定版本号重试次数)

基于最新版本号重试5次:
```
POST /test_index/_update/1?retry_on_conflict=5
{
    "doc": {
	"name": "小1王"
    }
}
```

#### 删除文档
```
DELETE /test_index/_doc/1?pretty

仅仅只是逻辑删除
当es数据越来越多的时候,es会在后台自动删除这些标记的数据
```
<!--more-->

#### 创建文档
```
默认操作是全量替换的
创建文档和全量替换的语法是一样的
,如果仅仅是想新建文档的话

PUT /index/_doc/id?op_type=create
或者
PUT /index/_doc/id/create

```

#### 查看指定id的文档
```
GET /test_index/_doc/1
```

#### 批量查询

```
//es下的批量查询
注意这个是 '查询的条件是es自带的属性'
GET /_mget
{
    "docs":[
     {
         "_index":"test_index",
         "_id":1
     },
      {
         "_index":"test_index",
         "_id":2
     }
     
    ]   
}

//索引下的批量查询
GET /test_index/_mget
{
    "docs":[
     {
         "_id":1
     },
      {
         "_id":2
     }
    ]
}
```
#### bulk批量增删改
bulk request会加载到内存里   
如果太大,吸能反而会下降,一般1000-5000条左右,大小在5-15M左右

语法如下:
```
{"action":{"metadata"}}
{"data"}
```
具体操作:  
需要注意的是每个json串时不能换行的 不然不通   
要按照`{"action":{"metadata"}}`这种方式写   
```java
POST /_bulk
//删除
{ "delete":{ "_index":"test_index", "_id":"3"}}
//创建
{ "create":{"_index":"test_index","_id":"3"}}
{ "name": "test12"}
//index操作:可以是创建文档，也可以是全量替换文档
{ "index":{ "_index":"test_index"}}
{"test_field":"replaced_test2"}
//更新
{ "update":{ "_index":"test_index", "_id":"3"}}
{"doc":{"name":"bulk test1"}}
```


# 搜索
```
took: 耗费了几毫秒
timed_out: 是否超时
_shards:数据拆成了5个分片,所以对于搜索请求,会打到所有的primary shard

hits.total:查询结果的数量
max_score: 搜索相关程度
hits.hits: 包含了匹配搜索的document的详细数据
```
##### 返回指定的字段
```
GET /index/document/1?_source=字段名,字段名2


_source选中要返回的字段
```

#### filter和query的对比
```
filter: 仅仅只是按照搜索条件过滤出需要的数据而已
query: 会去计算每个document相对于搜索条件的相关度,并按照相关度进行排序

一般来说,
如果你是在进行搜索,需要将最匹配搜索条件的数据先返回,那么用query;
如果你只是要根据一些条件筛选出一部分数据,不关注其排序,那么用filter;

## 性能对比
filter性能高 

filter: 不需要计算相关度分数,不需要按照相关度分数进行排序,同时还有内置的自动cache最常使用filter的功能

query:相反,要计算相关度分数,按照分数进行排序,而且无法cache结果



```


#### query DSL
简单来说 就是_search下面body包含一个{}
```
//查询所有的数据
GET /index/_search
{
    "query":{
        "match_all":{}
    }
}

//查询匹配属性
GET /index/_search
{
    "query":{
        "match":{
            "name":"yegao"
        }
    },
    "sort":[
       {
           "price":"desc"
       }
    ]
}

//查询指定数量
GET /index/_search
{
    "query":{
        "match_all":{}
    },
    "from":1,  //第几个商品开始查 从0开始计数
    "size":2  //查两个
}

//查询显示指定字段
GET /index/_search
{
    "query":{
        "match_all":{}
    },
    "_source":["name","price"]
}


//查询 不进行分词处理

GET /index/_search
{
    "query":{ 
       "term":{ //不进行分词查询
            "test_field":"test hello"
            }
    }
}

```
#### exist query (搜索的field不能为null)
```
GET /index/_search
{
    "query":{
        "exists":{
            "field": "title"
        }
    }
}

```

#### query filter (条件过滤)

```
GET /index/_search
{
    "query":{
        "bool":{
            "must":{ //精准匹配
                "match":{
                    "name":"yet"
                }
            },
            "filter":{
                "range":{ //范围过滤
                    "price":{"gt":25}
                }
            }
        }
    },
    "_source":["name","price"]
}
-------------------------------------
//如果只是要单纯的过滤的话
{
    "query":{
        "constant_score":{
            "filter":{
                "range":{
                    "age":{
                        "gte":30
                    }
                }
            }
        }
    }
}


```

#### bool的使用 多条件筛选数据
bool:  
must,must_not,should,filter

```
GET /index/_search
{
"query":{
{
    "bool":{
        "must": { //必须匹配的条件
        "match":{
            "title":"how to make millions"
        }
    },
      "must_not": { //必须不匹配的条件
        "match":{
            "tag":"pan"
        }
    },
    "should":[ //可能存在的条件
    { "match":{"tag":"starred"} }
    ],
    "filter":{ //过滤条件
         "range":{ //范围过滤
                    "price":{"gt":25}
                }
    }
}
}
}
```

#### full-text search  (全文搜索)
全文搜索
结果为: 存在A 或者B 或者A+B的  
匹配度最高的在搜索结果最上面

```
GET /index/_search
{
    "query":{
        "match":{
            "producer":"A B"
        }
    }
}

```

#### phrase search (短语搜索)
全文检索:对应是根据倒排索引进行一一匹配  相关度

短语搜索:必须都存在 两个字符串 才算结果
```
GET /index/_search
{
    "query":{
        "match_phrase":{
            "producer":"yagao producer"
        }
    }
}
```


#### highlight search (高亮搜索结果)
返回结果会分为两段
一段是高亮字段`yagao producer`会加上`em`高亮显示    

需要注意: 高亮的字段必须是查询的字段

```
GET /index/_search
{
    "query":{
        "match_phrase":{
            "producer":"yagao producer"
        }
    },
    "highlight":{
        "fields":{ //高亮字段
            "producer":{}
        }
    }
}
```

# 聚合分析
注意:聚合查询的话 需要分组字段必须为`fielddata:true`
```
可手动修改
PUT /index/_mapping/document
{
    "properties":{
        "tags":{
            "type":"text",
            "fielddata": true
        }
    }
}
```
#### 计算每个tag下的document数量
```
GET /index/_search
{
    "aggs":{ //聚合
        "all_tags":{ //给聚合一个名称 
            "terms":{ //按照指定的field进行分组 计算数量
                "field":"tags"
            }
        }
    }
}

```

#### 聚合查询 (不带出原有数据)
```
GET /index/_search
{   
    "size": 0, //不带出原有数据查询出来,只查聚合结果
    "aggs":{
        "all_tags":{ 
            "terms":{
                "field":"tags"
            }
        }
    }
}
```

#### 关键词查询后+分组

```
GET /index/_search
{
    "size": 0,
    "query":{
        "match":{
            "name":"yagao"
        }
    },
    "aggs":{
        "all_tags":{ 
            "terms":{
                "field":"tags"
            }
        }
    }
}

```

#### 分组查询+每组平均数 

```
GET /index/_search
{
    "size": 0,
    "aggs":{
        "all_tags"{ 
            "terms":{
                "field":"tags"
            },
            "aggs":{
                "avg_price":{
                    "avg":{ //求平均数
                        "field":"price"
                    }
                }
            }
        }
    }
}

```

#### 先分组->再根据内层的平均数->再降序


```
GET /test_index/_search
{
  "size": 0,
  "aggs": {
    "all_tags": {
      "terms": {
        "field": "tags",
        "order": {   //根据聚合结果,降序的内容先写在上面
          "avg_price": "desc"
        }
      },
      "aggs": {
        "avg_price": {//求平均数
          "avg": {
            "field": "age"
          }
        }
      }
    }
  }
}
```

#### 按照指定的区间进行分组 再每组内再按照tag进行分组,最后再计算每组的平均价格

这样会求出三段 0-20 20-40 40-60    三段内进行分组不干预其他数据
```
GET /test_index/_search
{
  "size": 0,
  "aggs": {
    "group_by_price": {
      "range": {
        "field": "age",
        "ranges": [ //获取出区间
          {
            "from": 0,
            "to": 20
          },
          {
            "from": 0,
            "to": 30
          },
          {
            "from": 2,
            "to": 25
          }
        ]
      },
      "aggs": { //进行分组
        "group_by_tags": {
          "terms": {
            "field": "tags"
          },
          "aggs": { //分组的同时计算平均数
            "avg_price": {
              "avg": {
                "field": "age"
              }
            }
          }
        }
      }
    }
  }
}
```

# 权重 boost参数

#### 基于boost查询细粒度控制

如果标题中包含 hadoop或elastisearch就优先搜索出来,  
boost参数(权重) 来控制优先级别 boost越小优先级越低

```
数据结构:
POST /test_index/_doc
{
  "name":"6",
  "age":18,
  "tags":["hadoop","java"]
}
```

```
GET /test_index/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "tags": "java"
          }
        }
      ],
      "should": [
        {"match": {
           "tags": {
             "query": "hadoop",
             "boost": 1
           }
        }},
        {"match": {
           "tags": {
             "query": "elasticsearch",
             "boost": 5
           }
        }}
      ]
    }
  }
```

# 利用best fields策略 自定义匹配

按照常规的匹配方式的话:  

是根据多个匹配的分数除以平均的,可能会导致我们想要的数据相关度分数不一样

![best fields策略](/img/2020-08-12/1.png)

```
//使用dis_max 取相关度最高的分数
get /test_index/_search
{
  "query": {
    "dis_max": {
      "queries": [
        {
          "match": {
            "title": "java solution"
          }
        },
        {
          "match": {
            "content": "java solution"
          }
        }
      ]
    }
  }
}
```





# 自定义搜索结果的排序规则

sort关键字

```
GET /index/_search
{
    "query":{
        "bool":{
            "filter":{"term":{"author_id":1 }}
               }
    },
"sort":{ //自定义排序规则 类似数据库order by
    "post_date":{ 字段名称
           "order":"desc"
        }
   }
}


```

# 排查执行计划 搜索是否合法

`GET /index/type/_validate/query?explain`  
body中就是查询条件  
可以判断查询语句是否合法 及其异常在哪里  

```
GET /test_index/_validate/query?explain
{
    "query":{
        "match":{
            "test_field":"test"
        }
    }
}

```

# 解决对某个field进行排序,结果可能不准确的问题
#### 发生原因
```
因为如果存在分词的话,会是多个单词,
再排序就不是我们想要的结果了
```

#### 解决方案
```
将一个String field建立两次索引,
一个分词,用来进行搜索;
一个不分词,用来排序

```
#### 写法如下
```
PUT /test_index/_mapping
{
  "properties": { 
    "tags": { 
      "type": "text",
      "analyzer": "ik_max_word",
      "fielddata":true, //开启正排索引
      "fields": {
        "raw": {  //作为全文搜索 不分词 用来聚合 和排序
          "type": "keyword",
          "ignore_above": 256
        }
      }
    },
    "name": {  //第二个字段
      "type": "text",
      "analyzer": "ik_max_word"
    }
  }
}

//查询:
GET /search
{
    "query":{
        "match":{
            "title":"elasticsearch"
        }
    },
    "sort":"title.raw"
}

```