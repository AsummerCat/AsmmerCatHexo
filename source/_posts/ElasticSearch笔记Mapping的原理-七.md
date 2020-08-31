---
title: ElasticSearch笔记Mapping的原理(七)
date: 2020-08-11 14:06:15
tags: [ElasticSearch笔记]
---

# 大致流程
```
(1)往es里面直接插入数据,es会自动建立索引,同时建立type医技对应的mapping


(2)mapping中就自动定义了每个field的数据类型


(3)不同的数据类型(比如说text和date),可能有的是exact value,有的是full text


(4) exact value,在建立倒排索引的时候,分词的时候,
是将整个值一起作为一个关键词建立到倒排索引中的,
full text,会经历各种各样的处理,
normalization(时态转换,同义词转换,大小写转换),分词等等.才会建立到倒排索引中


(5)同时呢,exact value和full text类型的field就决定了,
在一个搜索过来的时候,对exact value field或者是full value 进行搜索的行为也是不一样的,会跟建立倒排索引的行为保持一致,
比如:
exact value搜索的时候,就是直接按照整个值进行匹配的
full text query string,也会进行分词和normalization再去倒排索引中去搜索


(6)可以用es的dynamic mapping,让其自动建立mapping,包括自动设置数据类型,
也可以提前手动创建index和type的mapping,直接对各个field进行设置,

比如:数据类型, 索引行为,分词器等等

)

```
<!--more-->

# mapping的核心数据类型
注意:  
只能创建index时手动建立mapping,或者新增field mapping,但是不能update field mapping

```
string,
byte,
short,
integer,
long,
float,
double,
boolean,
date

```
### mapping复杂数据类型
```
1.multivalue field
可存放数组
{"tags":["tag1","tag2"]}

2. empty field
可以存放 null ,[] ,[null]

3.obejct field
可以存放对象
```

### 查看mapping
```
GET /index/_mapping/type

```

### dynamic mapping (es自动创建mapping)
如何自动映射  
```
true or false  --> boolean
123            --> long
123.45         --> double
2017-01-01     --> date
"hello word"   --> string


```

## 不同的Mapping可能导致分词器切分的内容不一致
```
比如: 时间 2017-01-01 会被拆分为
2017
01
01
```
```
普通查询(分词器拆分):

GET _search?q=2017-01-01
 会被拆分未 2017  01  01
 
---------------------------------

指定类型查询:
建立倒排索引

GET _search?q=post_date:2017-01-01

date,会作为exact value去建立索引
```


###  加入分词器

```
//测试分词器
GET /_analyze
{
    "analyzer":"standard",
    "text":"Text to analyze"
    //ik_max_word 这个是中文分词器
}

```

# 定制String类型数据如何建立索引及其分词 

## 手动建立mapping和分词器
#### 手动新建mapping
```
PUT /website?pretty  //创建索引

//定义mapping
PUT /website/_mapping
{
  "properties": {
    "tags": {
      "type": "text",
      "analyzer": "ik_max_word",
      "fielddata": true,
      "index": true, //使用分词器
      "fields": {
        "raw": {
          "type": "keyword",
          "ignore_above": 256
        }
      }
    },
    "name": {
      "type": "text",
      "analyzer": "ik_max_word"
    },
    "post_date": {
      "type": "date"
    },
    "title": {
      "type": "text"
    },
    "author_id": {
      "type": "long"
    }
  }
}
```
#### 新增设置某个字段关闭分词
```
analyzed: 指定分词
not_analyzed: 指的是不走关闭分词 查询走exact value
no: 不建立索引

PUT /website/_mapping/article
{
    "properties":{
        "tags":{
            "type": "text",
            "index": false //idnex选项控制字段的值是否被索引。
            接受trueorfalse，默认为true。该参数设置为false的字段是不能被搜索的。
        }
    }
}

```