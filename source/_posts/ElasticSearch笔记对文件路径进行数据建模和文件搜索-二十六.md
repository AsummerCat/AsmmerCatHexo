---
title: ElasticSearch笔记对文件路径进行数据建模和文件搜索(二十六)
date: 2020-08-25 23:23:48
tags: [ElasticSearch笔记]
---


# 首先创建一个 文件路径的分词器
这样的话 能切割文件路径
比如: `/home/java/1.8/jre`
```
能被分词为:
/home
/home/java
/home/java/1.8
/home/java/1.8/jre
```
<!--more-->

## 创建 文件路径分词器语法
```
PUT /fs   //创建索引(index)
{
    "settings":{
        "analysis":{
            "analyzer":{ //创建分词器
                "paths":{   //分词器名称
                    "tokenizer":"path_hierarchy"
                }
            }
        }
    }
}
'path_hierarchy'
这个就是专门用来对文件路径进行tree结构的拆分的
```

# 测试分词器语句
```
GET fs/_analyze
{
    "analyzer":"paths",
    "text":"/home/java/1.8/jre"
}
```
输出结果
```
{
  "tokens" : [
    {
      "token" : "/home",
      "start_offset" : 0,
      "end_offset" : 5,
      "type" : "word",
      "position" : 0
    },
    {
      "token" : "/home/java",
      "start_offset" : 0,
      "end_offset" : 10,
      "type" : "word",
      "position" : 0
    },
    {
      "token" : "/home/java/1.8",
      "start_offset" : 0,
      "end_offset" : 14,
      "type" : "word",
      "position" : 0
    },
    {
      "token" : "/home/java/1.8/jre",
      "start_offset" : 0,
      "end_offset" : 18,
      "type" : "word",
      "position" : 0
    }
  ]
}

```

# 使用案例

## 先创建一个含有路径的字段的mappings
```
PUT /fs/_mappings
{
  "properties": {
    "name": {
      "type": "keyword"
    },
    "path": {
      "type": "keyword",
      "fields": {
        "tree": {
          "type": "text",
          "analyzer": "paths"
        }
      }
    }
  }
}

PUT /fs/_mappings
{
  "properties": {
    "name": {
      "type": "text",
      "analyzer": "ik_max_word"
    },
    "path": {
      "type": "keyword",
      "fields": {
        "tree": {
          "type": "text",
          "analyzer": "paths"
        }
      }
    }
  }
}
```

## 创建doc
```
PUT /fs/_doc/1
{
  "name":"小明的文件夹",
  "path":"/home/index/java/1.8/私密文件夹"
}

```

## 对文件系统进行搜索
<font color="red">注意:进行搜索文件路径的时候需要用指定分词器为'path_hierarchy'这个的字段进行检索才能根据层级出现 ,其他的会搜索不出结果</font>
```
GET /fs/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "name": "小"
          }
        },
        {
          "constant_score": {
            "filter": {
              "term": {
                "path.tree": "/home"
                //注意这里需要搜索文件的层级 需要用刚刚分词器为'paths'的字段
              }
            },
            "boost": 1.2
          }
        }
      ]
    }
  }
}


```