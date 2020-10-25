---
title: Elaticsearch-简单使用二
date: 2020-02-14 20:49:45
tags: [elasticSearch笔记,SpringBoot]
---

# Elaticsearch

## 好处

```java
Elasticsearch不仅仅是Lucene和全文搜索引擎，它还提供：

分布式的实时文件存储，每个字段都被索引并可被搜索
实时分析的分布式搜索引擎
可以扩展到上百台服务器，处理PB级结构化或非结构化数据
  
  查询效率近乎实时
```

<!--more-->

## 概念

```
Elastic 本质上是一个分布式数据库，允许多台服务器协同工作，每台服务器可以运行多个 Elastic 实例。

单个 Elastic 实例称为一个节点（node）。一组节点构成一个集群（cluster）。
```



## 数据结构

```
Index :索引 ->等同于数据库的一张表

type: 类似于 多个type多个表结构 大致字段相等   比如按区域区分 厦门/北京 节点
根据规划，Elastic 6.x 版只允许每个 Index 包含一个 Type，7.x 版将会彻底移除 Type。
下面的命令可以列出每个 Index 所包含的 Type。
$ curl 'localhost:9200/_mapping?pretty=true'

Document:Index 里面单条的记录称为 Document（文档）。许多条 Document 构成了一个 Index。
        |同一个 Index 里面的 Document，不要求有相同的结构（scheme），但是最好保持相同，这样有利于提高搜索效率。
        
mapping: 维护了 Document的字段结构 需要注意的是 如果需要分词 需要在mapping中的字段设置

filed: Document中的字段
```



## API调用

#### 创建删除index

```
新建 Index，可以直接向 Elastic 服务器发出 PUT 请求。下面的例子是新建一个名叫weather的 Index。
$ curl -X PUT 'localhost:9200/weather'
服务器返回一个 JSON 对象，里面的acknowledged字段表示操作成功。
{
  "acknowledged":true,
  "shards_acknowledged":true
}

删除index: $ curl -X DELETE 'localhost:9200/weather'
```

#### 创建带分词的索引index

```
新建一个 Index，指定需要分词的字段。这一步根据数据结构而异，下面的命令只针对本文。基本上，凡是需要搜索的中文字段，都要单独设置一下。

$ curl -X PUT 'localhost:9200/accounts' -d '
{
  "mappings": {
    "person": {
      "properties": {
        "user": {
          "type": "text",
          "analyzer": "ik_max_word",
          "search_analyzer": "ik_max_word"
        },
        "title": {
          "type": "text",
          "analyzer": "ik_max_word",
          "search_analyzer": "ik_max_word"
        },
        "desc": {
          "type": "text",
          "analyzer": "ik_max_word",
          "search_analyzer": "ik_max_word"
        }
      }
    }
  }
}'
上面代码中，首先新建一个名称为accounts的 Index，里面有一个名称为person的 Type。person有三个字段。

user
title
desc
这三个字段都是中文，而且类型都是文本（text），所以需要指定中文分词器，不能使用默认的英文分词器。

Elastic 的分词器称为 analyzer。我们对每个字段指定分词器。

"user": {
  "type": "text",
  "analyzer": "ik_max_word",
  "search_analyzer": "ik_max_word"
}
上面代码中，analyzer是字段文本的分词器，search_analyzer是搜索词的分词器。ik_max_word分词器是插件ik提供的，可以对文本进行最大数量的分词。
```

#### 新增记录

```
$ curl -X PUT 'localhost:9200/accounts/person/1' -d '
{
  "user": "张三",
  "title": "工程师",
  "desc": "数据库管理"
}' 

服务器返回的 JSON 对象，会给出 Index、Type、Id、Version 等信息。
这边是 手动传入一个id  =1的记录
如果不指定id 会自动生成一个随机字符串
*** 还有一个需要注意的是: 如果index输入错误 会先自动生成一个index再写入

```

#### 查看记录

```
向/Index/Type/Id发出 GET 请求，就可以查看这条记录。

$ curl 'localhost:9200/accounts/person/1?pretty=true'
上面代码请求查看/accounts/person/1这条记录，URL 的参数pretty=true表示以易读的格式返回。
返回: "found" : true, 表示查询成功
"found" : false,表示查询失败 不存在id 或者id不正确
```

#### 删除记录

```
curl -X DELETE 'localhost:9200/accounts/person/1'
```

#### 更新记录

```
就是重新发送一遍插入请求 带上id
$ curl -X PUT 'localhost:9200/accounts/person/1' -d '
{
  "user": "张三",
  "title": "工程师",
  "desc": "数据库管理1111"
}' 
```

####  返回所有记录

```
$ curl 'localhost:9200/accounts/person/_search'
```

### 全文搜索

```
$ curl 'localhost:9200/accounts/person/_search'  -d '
{
  "query" : { "match" : { "desc" : "软件" }}
}'


Elastic 默认一次返回10条结果，可以通过size字段改变这个设置。
$ curl 'localhost:9200/accounts/person/_search'  -d '
{
  "query" : { "match" : { "desc" : "管理" }},
  "size": 1
}'
```

#### 逻辑运算

```
如果有多个搜索关键字， Elastic 认为它们是or关系。


$ curl 'localhost:9200/accounts/person/_search'  -d '
{
  "query" : { "match" : { "desc" : "软件 系统" }}
}'
上面代码搜索的是软件 or 系统。

如果要执行多个关键词的and搜索，必须使用布尔查询。


$ curl 'localhost:9200/accounts/person/_search'  -d '
{
  "query": {
    "bool": {
      "must": [
        { "match": { "desc": "软件" } },
        { "match": { "desc": "系统" } }
      ]
    }
  }
}'
```

