---
title: ElasticSearch笔记自定义dynamicMapping策略(十四)
date: 2020-08-11 14:11:22
tags: [ElasticSearch笔记]
---

# ElasticSearch笔记自定义dynamicMapping策略(十四)
## 目的
给自己的index设置默认的mapping生成,  
设置了自定义dynamic mapping策略  
可以有这种情况:  
你直接创建一个document里面存在没有定义过的mapping,
是可以直接根据自定义策略进行报错,或者忽略的  

<!--more-->

# 自定义dynamic mapping策略
三种策略
---
true:遇到陌生字段,就进行dynamic mapping  
false:遇到陌生字段,就忽略  
strict:遇到模式字段,就报错  
---

## 常规使用
```
PUT /my_index
{
    "mapping":{
        "my_type":{
            "dynamic": "strict"  //设置dynamic mapping策略 (全局的)
            "properties":{
                "title":{"type":"string"},
                "stash":{
                    "type": "object",
                    "dynamic":true (单个字段的)
                }
            }
        }
    }
}

```
使用   
例如:
```
PUT /my_index/my_type/1
{
    "title":"This doc adds a new field",
    "stash":{"new_field":"Success!"}
}
```

## 定制dynamic mapping策略
#### date_detection
默认会按照一定格式识别date,比如yyyy-MM-dd.但是如果某个field先过来的是`2017-01-01`的值,就会被自动`dynamic mapping`成date,后面如果再来一个`hello word`之类的值就会报错.  
可以手动关闭某个type的date_datection,如果有需要,自己手动指定某个field为date类型
```
PUT /my_index
{
    "mappings":{
        "my_type":{
        //手动关闭date_datection
            "date_datection":false
        }
    }
}
或者
PUT /my_index/_mapping/my_type
{
        "date_datection":false  
}
```
#### 自定义自己的dynamic mapping template(type level)
```
PUT /my_index
{
    "mappings":{
        "my_type":{  
         //自定义策略
            "dynamic_templates":[
            {"es":{
                //如果匹配到*_es这种格式的document就是用当前策略
                "match": "*_es",
                "match_mapping_type":"string",
                "mapping":{
                    "type": "string",
                    "analyzer": "spanish"
                }
            }},
            {"en":{
            //如果匹配到*这种格式的document就是用当前策略
                    "match": "*",
                    "match_mapping_type":"string",
                    "mapping":{
                        "type":"string",
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

#### 定制自己的default mapping template (index level)
```
PUT /my_index
{
    "mappings":{
        "_default":{
        //全局关闭_all这个搜索
            "_all":{"enabled": false}
        },
        "blog":{
        //指定字段开启
            "_all":{"enabled": true}
        }
    }
}
```