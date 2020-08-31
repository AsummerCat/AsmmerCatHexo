---
title: ElasticSearch笔记使用动态映射模板定制映射策略(三十二)
date: 2020-08-31 22:01:09
tags: [ElasticSearch笔记]
---

## 使用场景
比如说,我么能来没有某个field,但是希望在插入数据的时候,es自动为我们做一个识别,动态映射出这个type的mapping,包括每个field的数据


<!--more-->

# 动态映射模板
1.  第一种  根据新加入的field的默认的数据类型,来进行匹配,匹配上某个预定义的模板;
2.  第二种  根据新加入field的名字,去匹配预定义的名字,或者去匹配一个预定义的通配符,然后匹配上某个预定义的模板


## 第一种 根据类型匹配映射模板 
```
PUT my_index
{
    "mappings":{
        "dynamic_templates":[
        {
            "integers":{  //自定义的模板名字
                "match_mapping_type":"long", //模板要匹配的类型
                "mapping":{
                    "type":"integer" //更新后的类型
                }
            }
        },
        {
            "text":{ /自定义的模板名字
                "match_mappin_type":"text", //模板要匹配的类型
                "mapping":{  //更新后的类型
                    "type":"text",
                    "fields":{
                        "raw":{
                            "type":"keyword",
                            "ignore_above":256
                        }
                    }
                }
            }
        }
        ]
    }
}

预先设定一个模板,然后插入数据的时候,相关的field就会根据我们的规则,来创建对应类型的字段

```

##  第二种 根据field的名字进行通配符匹配映射
```
PUT /my_index
{
    "mappings":{
        "dynamic_templates":{
            "longs_as_strings":{  //自定义名称
                "match_mapping_type":"text", //匹配原类型
                "match": "long_*", //匹配long_打头的数据
                "unmatch":"*_text",//过滤掉_text结尾的数据
                "mapping":{
                    "type":"long"
                }
            }
        }
    }
}


```