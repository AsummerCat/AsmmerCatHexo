---
title: ElasticSearch笔记修改分词器及其定制自己的分词器(十二)
date: 2020-08-11 14:09:57
tags: [ElasticSearch笔记]
---

# ElasticSearch笔记修改分词器及其定制自己的分词器(十二)

## 默认的分词器
->>>`standard`  

```
standard tokenizer :以单词边界进行切分

standard token filter : 什么都不做

lowercase token filter :将所有字符转换为小写

stop token filer: (默认被禁用)移除停用词,比如 a the it 等等
```
<!--more-->

## 修改分词器的设置
启用english移除停用词 `stop token filer`
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
#### 测试修改后的分词器 
```
GET /my_index/_analyze
{
  "analyzer":"es_std",
  "text":"a dog is in the house"
}
这样会分出两个词 dog 和house 其他都会忽略
```

# 定制化自己的分词器
```
PUT  /my_index
{
    "settings":{
        "analysis":{
            "char_filter":{ //自定义转换字符
                "&_to_and":{ //自定义转换字符的名称
                    "type": "mapping",
                    //自定义转换字符的规则
                    "mappings":["&=> and "]
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
                "char_filter":["html_strip","^_to_and"],
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
#### 测试自定义分词器
```
GET /my_index/_analyze
{
    "text":"The quick & brown fox",
    "analyzer": "my_analyzer"
}

```

#### 将自定义分词器设置给字段
```
PUT /my_inedx/_mapping/my_type
{
    "properties":{
        "title":{
            "type": "string",
            "analyzer": "my_analyzer"
        }
    }
}

```