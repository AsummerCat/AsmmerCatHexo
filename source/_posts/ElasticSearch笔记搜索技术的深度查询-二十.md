---
title: ElasticSearch笔记搜索技术的深度查询(二十)
date: 2020-08-23 14:48:49
tags: [ElasticSearch笔记]
---

# ElasticSearch笔记深度揭秘搜索技术新特性(二十)

# 权重 boost参数

#### 基于boost查询细粒度控制

如果标题中包含 hadoop或elastisearch就优先搜索出来,  
boost参数(权重) 来控制优先级别 boost越小优先级越低
<!--more-->

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

# 利用best fields策略 自定义匹配 dis_max
这个策略是……。
主要是将某个
field匹配尽可能多的关键词的_doc优先返回回来。 

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

# 基于tie_breaker参数优化dis_max参数效果
1. dis_max只取某一个query最大的分数
2. 使用tie_breaker将其他的分数也考虑进去

tie_breaker参数的意义:
```
在于说,将其他query的分数,乘以tie_breaker,
然后综合与最高分数的那个query的分数,综合在一起计算
```
### 写法
```

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
      ,
      "tie_breaker": 0.7
    }
  }
}

```

# minimum_should_match  去长尾
也就说，   
比如你搜索5个关键字，很多结果都是只匹配到1个关键字的。  
与你想要的结果相差甚远，这些结果就是长尾。   
> minimum_should_match=40%   也就是最少需要匹配到2个

控制搜索结果匹配度，只有匹配到一定数量的关键词的数据，才能返回

> 搜索中加入

>   "minimum_should_match":"30%"

语法     
```
get /test_index/_search
{
    "query":{
        "multi_match":{
            "field":「"title^2"，"age"」 ////title字段添加权重为2 
            "type": "best_fields",
            "minimum_should_match":"30%"
            //两个关键字最少匹配为30%  
        }
    }
   
}

```


#  利用most_fields策略 
尽可能的返回更多的fields
匹配到某个关键词的_doc，优先返回回来
```
get /test_index/_search
{
    "query":{
        "multi_match":{
            "query":"learning coursee", //需要查询的数据
            "type":"most_fields", //搜索的类型
            "fields":["sub_title","sub_title.std"] //需要匹配的字段
    }
}

sub_title.std 这个为新的分词器结构
```
与best_fields的区别
```
1. best_fields,是对多个field进行搜索,跳远某个field匹配度最高的那个分数,
同时在多个query最高分相同的情况下,在一定程度上考虑其他query的分数.
  简单来说,你对多个field进行搜索,就想搜索到某个field尽可能包含更多关键字的数据
  
  优点:
  通过best_fields策略,以及综合考虑其他field,还有minimum_should_match支持,
  可以尽可能精准的将匹配的结果推送到最前面
  
  缺点:
  除了那些精准匹配的结果,其他差不多大的结果,排列结果不是太均匀
  
  例如: 百度->最匹配的最前面,但是其他的就没什么区分度了
  
  
 2. most_fields, 综合多个field一起进行搜索,尽可能多地让所有field的query参与到总分数的计算中来,
 此时就会是个大杂烩,出现结果不一定精准,  
 
某一个document的一个field包含更多的关键字,但是因为其他的document有更多field匹配到了,所以排在前面;
所以需要建立sub_title.std这样的field,尽可能让某一个field精准匹配query string,贡献更高的分数,将跟精准匹配的数据排在前面

优点:
  将尽可能匹配更多field的结果推送到最前面,整个排序结果是比较均匀的
缺点:
  可能那些精准匹配的结果,无法推送到最前面

例如:
  明显的most_fields策略,搜索结果比较均匀,但是的确要翻好几页才能找到最匹配的结果
```

##  利用most_fields策略 解决cross-fields问题
指的是:一个唯一标识,跨多个field.  
比如一个人,标识,是地址,名称.   
名称可以散落在first_name和last_name中,地址可以散落在country,province和city中
> 以上的情况可能会导致 搜索相关度不精准

再搭配 `copy_to`定制组合field解决`cross-fields`搜索弊端
### 先构建一个cpoy-to的整合字段
```
PUT /index/_mapping/
{
    "properties":{
        "new_author_first_name":{
            "type":"text",
            "copy_to":"new_author_full_name" //拷贝给新字段
        },
           "new_author_last_name":{
             "type": "text",
             "copy": "new_author_full_name"
           },
          "new_author_full_name":{
              "type": "text"
          }
    }
}

//这样的话 就会多一个整合的字段 使用copy_to

```
## 或者使用原生的  cross_fields来解决 提高准确度 (推荐)
这样效果:  
learning 必须出现在`author_first_name`或`author_last_name`  
coursee 必须出现在`author_first_name`或`author_last_name`
```
get /test_index/_search
{
    "query":{
        "multi_match":{
            "query":"learning coursee", //需要查询的数据
            "type":"cross_fields", //搜索的类型
            "operator":"and",
            "fields":["author_first_name","author_last_name"] //需要匹配的字段
    }
}
```

# 短语搜索和近似匹配    match_phrase

//先搜索出结果,再根据结果进行近似匹配,然后重计分

match: 只要简单的匹配到一个term,就可以理解将trem对应的doc作为结果返回,扫描倒排索引,扫描到了就ok

phrase match: 首先扫描到所有term的doc list;  
找到包含所有term的doc list;    
然后对每个doc都计算每个trem的position,是否符合指定的范围;  
slop,需要进行复杂的计算,来判断是否通过slop移动,匹配一个doc

match 比 phrase match性能高10倍    (短语搜索)   
match 比 phrase match+slop性能高20倍   (近似匹配)



### 短语搜索
这样能把`yagao`和`producer`靠的很近的document优先返回

如果用`match`的话 仅仅只能查询到包含这两个的document
```
//两个单词靠在一起
GET /index/_search
{
    "query":{
        "match_phrase":{
            "producer":"yagao producer"
        }
    }
}

```

### 近似匹配 
当然我们可以使用精确匹配match_phrase来解决这个问题，但是又太过严格，因为我们可能搜索 ' 华为大屏手机 ',我们能够通过使用 slop 参数将灵活度引入短语匹配中。
```
//两个单词靠的越近,doc分数越高

slop 参数告诉 match_phrase 分词后的词条项中的查询词条相隔多远时仍然能将文档视为匹配。
也可以说查询词条和文档中的词条相差slop个词条就可以匹配。
当然查询词的词条相差的越少，则该文档的评分会越高。

GET /index/_search
{
    "query": {
        "match_phrase": {
            "title": {
                "query": "华为手机",
                "slop":  50
            }
        }
    }
}

```

#### 还可以提高召回率和精准度
```
GET /index/_search
{
    "query":{
        "bool":{
            "must":{
             "match":{
                 "title":{
                     "query":"java spark",
                     "minimum_should_match":"50%" //最少必须匹配1个
                 }
             }
          },
          "should":{
              "match_phrase":{ //近似匹配
                  "title":{
                      "query":"java spark",
                      "slop": 50   //两个词条相差多远
                  }
              }
          }
        }
        
    }
}

```
#### 使用rescore_query 来重计分 提高优化近似匹配
因为用户大部分是分页搜索  
最多前几页 一页10条 看5页

所以我们可以去针对前50个doc进行slop移动去匹配,贡献自己的分数即可,不需要对全部1000个doc都去进行计算和贡献分数
```
GET /index/_search
{
    "query":{
        "match":{  //只要简单的匹配到一个trem 作为结果返回(扫描倒排索引)
            "title":{
                "query":"java spark",
                "minimun_should_match": "50%"
            }
        }
    },
    "rescore":{  重计分
        "window_size":50, //选择搜索结果的前50个进行重计分
        "query":{
            "rescore_query":{
                "match_phrase":{
                   "title":{
                       "query": "java spark",
                       "slop": 50
                   }
            }
        }
    }
}
```

# 前缀搜索  - 通配符搜索 -  正则搜索
### 前缀搜索 prefix
不计算相关性评分  
前缀越短,要处理的doc越多,性能越差,尽可能用长前缀搜索

字段要为 keyword 才好匹配
```
get /index/_search
{
    "query":{
        "prefix":{
            "title":{ //查询的字段
                "value":"C4"
            }
        }
    }
}

```

### 通配符搜索  wildcard
跟前缀搜索类似,功能更加强大
```
get /index/_search
{
    "query":{
       "wildcard": {
            "postcode": {
                "value":"W?F*HW"
            }
        }
    }
}

?: 任意子弹
*: 0个或任意多个字符
```

### 正则搜索 regexp
```
get /index/_search
{
    "query":{
        "regexp":{
            "titles":"W[0-9].+"
        }
    }
}

[0-9]: 指定范围内的数字
[a-z]: 指定范围内的字母
.: 一个字符
+: 前面的正则表达式可以出现一次或多次

```

# 实现边输入边提示 搜索推荐 低性能

1.   原理跟match_phrase类似,唯一的区别,就是最后一个trem作为前缀去搜索
2.   也可以指定slop,但是只有最后一个term会作为前缀
3.   max_expansions:指定最大匹配多少个term,超过这个数量就不继续匹配了,限定性能

```
GET /index/_search
{
   "query":{
        "match_phrase_prefix":{
           "title:{
               "query": walker jon b",
               "slop": 10,
               "max_expansions":50
      }
   }
}

'walker'就是去进行match,搜索对应的doc
`w`,会作为前缀,去扫描整个倒排索引,找到所有w开头的doc,
即包含hello,又包含w开头的字符的doc
根据你的slop去计算,看在slop范围内,能不能让'hello w',正好跟doc中的hello和w开头的单次的postion相匹配

max_expansions控制匹配的trem 不再继续搜索倒排索引了
'ps'尽量不要用,因为前缀搜索'w' 还是会倒排索引的搜索
```
# 实现边输入边提示 搜索推荐 高性能
基于 `ngram分词`
比如: nginx  
可以根据不同的拆分规则拆分为: 
```
ngram length=1  n g i n x
ngram length=2  ng gi in nx
ngram length=3  ngi gin inx
```
原理是:创建index的时候就建立了 而不用在搜索的时候再去根据一个前缀扫描整个倒排索引.
而是直接拿前缀去匹配倒排索引,匹配上就好了 
### 写法
创建索引
```
PUT /my_index
{
    "settings":{
        "analysis":{
            "autocomplete_filter":{ //创建一个过滤器
                "type":"edge_ngram",
                "min_gram":1, //gram分词 单个词条最短1
                "max_gram":20  //gram分词 单个词条最长20
            }
        },
        "analyzer":{  //创建自定义分词器
            "autocomplete":{
                "type":"custom",
                "tokenizer":"standard",
                "filter":[ //引用过滤器
                "lowercase",
                "autocomplete_filter"
                ]
            }
        }
    }
}

注意:搜索的时候字段的分词器还是使用标准的
创建字段
PUT /my_index/_mapping/
{
    "properties":{
        "title":{
            "type":"text",
            "analyzer": "autocomplete", //gram分词
            "search_analyzer":"standard" //搜索时候的分词器
            
        }
    }
}

```

# 降低搜索分数
`negative`
```
GET /index/_search
{
    "query":{
        "boosting":{
            "positing":{
                "match":{
                    "text":"apple"
                }
            }
        },
        "negative":{
            "match":{
                "text":"pie tart tree"
            }
        },
        "negative_boost": 0.5
    }
}

意思也就是 
如果在搜索结果里 有匹配到'negative'中的pie,tart,tree的话 相关度分数减半 也就是 *0.5
以起到降低分数的作用
```

# 不计算相关度评分

加入'constant_score' 一般配合fiter使用

所有的doc相关度都是1

```java
GET /index/_search
{
    "query":{
        "bool":{
            "should":[
            {
                "constant_score":{
                    "query":{
                        "match":{
                            "title":"java"
                        }
                    }
                }
            },
            {
                "constant_score":{
                    "query":{
                        "match":{
                            "title":"spark"
                        }
                    }
                }
            }
            ]
        }
    }
}
```

