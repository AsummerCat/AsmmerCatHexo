---
title: ElasticSearch笔记fielddata核心原理(二十五)
date: 2020-08-25 23:23:08
tags: [elasticSearch笔记]
---

# fielddata 核心原理
fieddata加载到内存的过程是懒加载的.对一个analzyed field执行聚合时,才会加载,而且是field-level加载的  
一个index的一个field,所有doc都会被加载,而不是少数doc  
`不是index-time创建,是query-time创建的`

<font color="red">
大意就是创建索引的时候不会生成fielddata,只有在查询的时候会生成fielddata.  
并且生成fielddata的时候是整个index的所有doc都进行创建fielddata,并且都加载到内存中
</font>

<!--more-->
```
注意: 
doc_value : 这个是创建index时候,建立的正排索引,在磁盘上
fielddata : 这个是搜索时,创建的正排索引,在内存上

```

# fielddata内存限制
```
在/es/config/elasticsearch.yml里面设置

indices.fielddata.cache.size: 20%
超出限制,清除内存已有的fielddata数据
```
默认无限制,  
可以限制内存使用,但是会导致频繁evict和reload,大量IO性能消耗,以及内存碎片和GC   
<font color="red">
因为会清除已有的数据,然后再加载
</font>

# 监控fielddata内存使用
```
GET /_stats/fielddata?fields=*
全局占用情况

GET /_nodes/stats/indices/fielddata?fields=*
每个node的占用情况

GET /_nodes/stats/indices/fielddata?level=indices&fields=*
每个索引的内存的占用情况
```


# circuit breaker 短路器的使用
如果一次`query load`的fielddata超过总内存,就会OOM  

circuit breaker 会估算query要加在的fielddata大小,如果超出总内存,就短路,query直接失败
```
在/es/config/elasticsearch.yml里面设置

indices.breaker.fielddata.limit: fielddata的内存限制,默认60%

indices.breaker.request.limit:执行聚合的内存限制,默认40%

indices.breaker.total.limit: 综合上面两个,限制在70%内
```

# _fielddata filter的细粒度内存加载控制
建立`_mappings`的时候,可以指定正排索引的相关参数,来进行控制内存的颗粒度

<font color="red">一般不会去设置这两个东西</font>
```
put /my_index/_mapping/
{
    "properties":{
        "tag":{
            "type":"text",
            "fielddata":{ //细粒度控制
                "filter":{
                    "frequency":{
                        "min": 0.01,
                        "min_segment_size": 500
                    }
                }
            }
        }
    }
}

'min': 仅仅加载至少在1%的doc中出现过的term对应的fielddata
比如说: hello,总共有1000个doc,hello必须在10个doc中出现,那么hello对应的fielddata才会加载到内存中来


'min_segment_size': 少于500的doc的segment(写入原理里的数据片,会合并的那个东西)不加载fielddata


```

# _fielddata预加载机制及其序号标记预加载

如果要对分词的field执行聚合,那么每次query的时候才会进行加载fielddata到内存中来,在创建field的时候,让fielddata预先创建.

也是就是说在建立倒排索引的时候,会同步生成fielddata的数据,这样field的聚合执行性能就大幅度增强来

## 预加载
```
PUT /my_index/_mapping
{
    "tags":{
        "type":"text",
        "fielddata":{
            "loading":"eager"
        }
    }
}


loading: xxx
```
## 序号标记预加载 (全局标记预加载 ->fielddata变为序号)
有很多重复值的情况下,会进行global ordinal标记

类似于相同内容 成组标记为 `0 1 2`  
建立的fielddata也会是这样子的,好处是减少重复字符串出现的次数,减少内存消耗

```
PUT /my_index/_mapping
{
    "tags":{
        "type":"text",
        "fielddata":{
            "loading":"eager_global_ordinals"
        }
    }
}
```

# 最后描述
## 注意
一般我们不对分词的field进行fielddata操作,
而是使用doc values   
语法:
```
默认情况下是自动开启的
PUT /my_index/_mappings
{
    "tag":{
        "type":"text",
        "doc_values":true //开启doc values
    }
}
```