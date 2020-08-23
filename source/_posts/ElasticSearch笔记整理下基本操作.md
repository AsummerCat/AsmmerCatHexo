---
title: ElasticSearch笔记整理下基本操作
date: 2020-08-11 13:56:55
tags: [ElasticSearch笔记]
---

# ElasticSearch笔记整理下基本操作
针对7.0以上的es版本
# 查看集群相关,健康之类的

###  快速检查集群健康状态
```
GET  /_cat/health?v
```
### 查看es的健康状态
```
get _cluster/health
```
<!--more-->

# index索引相关
### 快速查看集群中有哪些索引
```
GET /_cat/indices?v
```
### 创建索引
```
PUT /test_index?pretty
```
### 删除索引
```
delete /test_index?pretty

DELETE /my_index
DELETE /index_one,idnex_two
DELETE /index_*
DELETE /_all
```

# document文档相关
es7.0以后没有type的概念,统一由`_doc`代替

### 路由插入数据
```
一个index上的数据会被封为多片,每片都在一个shard中

PUT /test_index/_doc/1?routing=age
{
  "name":"小明",
  "age":1
}

```
### 插入文档
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
### 更新文档 (全量替换)
语句跟插入相同
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
### 更新文档 (部分字段更新)
更新文档指定字段:
```
POST /test_index/_update/1
{
    "doc": {
    	"age":19
    }
}
```
### 更新文档 (基于最新版本号重试次数)
基于最新版本号重试5次:
```
POST /test_index/_update/1?retry_on_conflict=5
{
    "doc": {
	"name": "小1王"
    }
}
```

### 更新文档 (基于乐观锁)
```
POST /test_index/_update/1?version=5
{
    "doc": {
    	"name":"111"
    }
}


原乐观锁后加上 &vesion_type=external

跟原来es的vesion的差别是:
  自定义version>es的version的时候才能完成修改
POST /test_index/_update/1?version=3&vesion_type=external
{
    "doc": {
	"name": "小1王"
    }
}

```
### 删除文档
```
DELETE /test_index/_doc/1?pretty
```

### 查看指定id的文档
```
GET /test_index/_doc/1
```

# 搜索相关
### 搜索超时设置
```
指的是单个shard的超时
搜索的时候会查找多个shard

GET /_search?timeout=30ms

```
### 基本语法
```
GET 
/_search                            所有索引,所有type下的所有数据都搜索出来
/index1/_search                     指定一个index,搜索其下所有type的数据
/index1,index2/_search              同时搜索两个index下的所有数据
/*1,*2/_search                      根据通配符去匹配多个index的所有数据
/index1/type1/_search               搜索index下指定type下的所有数据
/index1/type1,type2/_search         搜索index下多个type下的所有数据
/index1,index2/type1,type2/_search  搜索多个index下多个type下的所有数据
/_all/type1,type2/_search           _all 标识搜索所有index下的指定type
```

### 查询index所有document
```
GET /test_index/_search
{
  "query": {
    "match_all": {}
  }
}

```

### 分页搜索
```
size:查询几条数据    (分页)
from:从第几条数据开始查
to: 到第几条数据为止


GET /_search?size=10     查询10条记录

GET /_search?size=10&from=0   从0开始查询10条

GET /_search?size=10&from=20  从20开始查询10条

```
### 批量查询
注意这个是 '查询的条件是es自带的属性'
```
GET /_mget
{
  "docs": [
    {
      "_index": "test_index",
      "_id": "1"
    },
    {
      "_index": "test_index",
      "_id": "2"
    }
  ]
}


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

### bulk批量处理数据
```
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
### 查询所有的数据
```
GET /index/_search
{
    "query":{
        "match_all":{}
    }
}
```
### 查询匹配属性
```
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

```
### range查询区间 
range: 用来做范围查询
filter.range.name
```
GET /index/_search
{
    "query":{
        "constant_score":{
        "filter":{
            "range":{
                "name":{
                    "gte":10,
                    "lte":20
                }
            }
        }
        }
    },
    "sort":[
       {
           "price":"desc"
       }
    ]
}

```

### 查询时间区间
日期在最近一个月内的
```
GET /index/_search
{
    "query":{
        "constant_score":{
        "filter":{
            "range":{
                "create_time":{
                    "gt":"now-1m"
                }
            }
        }
        }
    }
}

或者指定日期减30天
GET /index/_search
{
    "query":{
        "constant_score":{
        "filter":{
            "range":{
                "create_time":{
                    "gt":"2017-01-01||-30d"
                }
            }
        }
        }
    }
}

```

### 查询指定数量的document
```
GET /test_index/_search
{
    "query":{
        "match_all":{}
    },
    "from":0,  
    "size":2 
}
```
### 查询显示指定字段
注意如果符合条件的记录 字段为null 也会返回该记录
```
GET /test_index/_search
{
    "query":{
        "match_all":{}
    },
    "_source":["name","age"]
}
```

### 查询 不对搜索内容进行分词处理
```
GET /index/_search
{
    "query":{ 
       "term":{ //不进行分词查询
            "test_field":"test hello"
            }
    }
}

GET /test_index/_search
{
    "query":{ 
       "terms":{ //批量查询两个关键词
            "name":["小","1"]
            }
    }
}
```

### exist query (搜索的field不能为null)
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

### query filter (条件过滤)
注意filter不能直接用在query上  
需要包裹:`"constant_score":{}`  
或者使用 'bool'
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

### bool的使用 多条件筛选数据
```
GET /test_index/_search
{
  "query": {
    "bool":{
        "must": {  //必须匹配的条件
          "match":{"name":"小" }
              },  
        "must_not": { //必须不匹配的条件
        "match":{"name":"1"}
                },
        "should":[ //可能存在的条件
    { "match":{"name":"王"} }
    ],
        "filter":{ 
         "range":{ "age":{"lt":25} }
            }
    }
  }
}
```

### full-text search  (全文搜索)
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

### phrase search (短语搜索)
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

### highlight search (高亮搜索结果)
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
# 权重 boost参数
### 基于boost查询细粒度控制
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

# 聚合分析
注意:聚合查询的话 需要分组字段必须为`fielddata:true`
```
可手动修改
PUT /index/_mapping
{
    "properties":{
        "tags":{
            "type":"text",
            //开启正排索引
            "fielddata": true
        }
    }
}
```
### 计算每个tag下的document数量
```
GET /index/document/_search
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
### 聚合查询(不带出原有数据)
```
GET /index/document/_search
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

### 关键词查询后+分组
```
GET /index/document/_search
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

### 分组查询+每组平均数 
```
GET /test_index/_search
{
    "size": 0,
    "aggs":{
         "test":{
              "terms":{ //分组查询
                "field":"tags"
            },
            "aggs":{
              "avf_age":{//根据分组的结果,在进行求平均数
                    "avg":{ 
                        "field":"age"
                    }
            }
            }
         }
    }
}
```

### 先分组->再根据内层的平均数->再降序
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
        "avg_price": {
          "avg": {
            "field": "age"
          }
        }
      }
    }
  }
}

```

### 按照指定的区间进行分组 再每组内再按照tag进行分组,最后再计算每组的平均价格
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

### 自定义搜索结果的排序规则
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

### 排查执行计划 搜索是否合法
`GET /index/type/_validate/query?explain`  
body中就是查询条件  
可以判断查询语句是否合法 及其异常在哪里  
```
GET /test_index/text_type/_validate/query?explain
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
      "fielddata": true,
      "analyzer": "ik_max_word",
      "fields": {
        "raw": { //不分词的索引
          "type": "keyword",
        }
      }
    },
    "name": {
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

# scoll滚动搜索(常用)
使用scoll滚动搜索,可以先搜索一批数据没然后下次再搜索一批数据,以此类推,知道搜索出全部的数据来.  
每次发送scroll请求,我们还需要指定一个scoll参数,指定一个时间窗口,每次搜索请求只要在这个时间窗口内能完成就可以了
### 语法
```
GET /index/_search?scroll=1m
{
    "query":{"match_all":{}},
    "sort":["_doc"], //排序
    "size": 1000, //单次查询数据条数
}
获取的结果会有一个scoll_id,下一次在发送scoll请求的时候,必须带上这个scoll_id


//下一次请求 只要指定个时间窗口参数就可以继续搜索了
这个语法是固定的 不需要加上index和type,这些都指定在了原始的 search 请求中。
GET /_search/scroll
{
    "scroll":"1m",
    "_scroll_id":"scoll_id的内容"
}

sizze会发送给每个shard,因此每次最多会返回size * primary shard
条数据
```


# 分词器
### 测试分词器
`ik_max_word`开源的中文分词器
```
GET /_analyze
{
    "analyzer":"standard",
    "text":"Text to analyze"
    //ik_max_word 这个是中文分词器
}

```

### 修改分词器的设置
```
PUT /my_index
{
    "settings":{
        "analysis":{
            "analyzer":{ //设置分词器
                "es_std":{ //分词器名称
                    "type": "standard",
                    //启用移除停用词
                    "stopwords":"_english_"
                }
            }
        }
    }
}
```
### 创建自定义分词器
```
PUT  /my_index
{
    "settings":{
        "analysis":{
            "char_filter":{ //自定义转换字符
                "&_to_and":{ //自定义转换字符的名称
                    "type": "mapping",
                    //自定义转换字符的规则
                    "mappings":["&=>and"]
                }
            },
            "filter":{ //配置停义词
                "my_stopwords":{
                    "type":"stop",
                    //忽略这两个词条
                    "stopwords":["the","a"]
                }
            },
            "analyzer":{  //自定义的分词器
                "my_analyzer":{ //自定义分词器的名称
                // 分词器为自定义用户创建
                    "type":"custom",
                //自定义转换字符  
                "char_filter":["html_strip","&_to_and"],
                //基于默认的分词器
                    "tokenizer": "standard",
                //配置自定义停义词   
                    "filter": ["lowercase","my_stopwords"]
                }
            }
        }
    }
}
```
### 将自定义分词器设置给字段
```
PUT /my_index/_mapping
{
    "properties":{
        "title":{
            "type": "string",
            "analyzer": "my_analyzer"
        }
    }
}
```


# 自定义dynamicMapping策略
(创建字段,根据该策略自动匹配规则)
### 创建自定义策略
```
//创建索引
PUT /my_index?pretty
//创建自定义mappings策略
PUT /my_index/_mappings
{
            "dynamic": "strict",//设置dynamic mapping策略 (全局的)
            "properties":{
                "title":{"type":"text"},
                "stash":{
                    "type": "object",
                    "dynamic":true //(单个字段的)
                }
            }
}

使用:
PUT /my_index/1
{
    "title":"This doc adds a new field",
    "stash":{"new_field":"Success!"}
}
这样创建stash 都是按照这个规则去生成的
```
### 自定义策略 避免自动识别日期
不设置会导致  
`2017-01-01`默认会变为date类型,后期无法插入字符串
```
手动将改字段设置为text

PUT /my_index/_mappings
{
            "properties":{
                "create_time":{"type":"text"}
            }
}
```

### 自定义策略 哪种字段按照哪种规则生成mapping
```
PUT /my_index
{
    "mappings":{
         //自定义策略
            "dynamic_templates":[
            {"es":{
                //如果匹配到*_es这种格式的document就是用当前策略
                "match": "*_es",
                "match_mapping_type":"string",
                "mapping":{
                    "type": "text",
                    "analyzer": "spanish"
                }
            }},
            {"en":{
            //如果匹配到*这种格式的document就是用当前策略
                    "match": "*",
                    "match_mapping_type":"string",
                    "mapping":{
                        "type":"text",
                        "analyzer":"english"
                    }
             } }

            ]
        }
    }
}


ps:
比如按照上面规则 aaa_es字段会进入es这个dynamic模板,
xxx字段会进入en这个dynamic这个模板

注意了 如果没有匹配到任何dynamic模板,默认就是standard分词器,会进入倒排索引


```

# document写入的优化内容
以下内容一般不修改
### 手动刷新到一个segment file中 进入os cache
```
POST /new_index/_refresh   
```
### 创建索引的时候设置自动refresh时间
默认是一秒refresh一次
```
PUT /new_index
{
    "settings":{
        "refresh_interval":"30s"
    }
}
```
### 设置异步translog的时间
```
PUT /new_index/_settings
{
    "index.translog.durability":"async", //异步执行预写预写日志
    "index.translog.sync_interval": "5s"
    //同步间隔 5秒一次
}
```
### 手动flush 写入磁盘 
清空预写日志和刷入磁盘
```
POST /new_index/_flush
```