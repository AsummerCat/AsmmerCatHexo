---
title: ElasticSearch笔记搜索结果highlight的详解(二十九)
date: 2020-08-31 21:58:44
tags: [ElasticSearch笔记]
---

## 普通写法
```
GET /my_index/_search
{
    "query":{
        "match":{
            "my_field":"elasticsearch"
        }
    },
    "highlight":{
        "fields":{
            "my_field":{}
        }
    }
}

```

<!--more-->

# 三种highlight介绍
1. plain highlight   底层:lucene highlight 默认的
2. posting highlight +index_options=offsets
3. fast vector highlight


##  第二种 posting highlight+index_options=offsets
1. 性能比plain highlight要高,因为不需要重新对高亮文本进行分词
2. 对磁盘的消耗更少
3. 将文本切割为句子,并且对句子进行高亮,效果更好

如果设置了`index_options`就是` posting highlight`模式

### 如何设置
```
mapping设置

PUT /my_index
{
    "mappings":{
        "properties":{
            "content":{
                "type":"text",
                "analyzer":"ik_max_word",
                "index_options":"offsets"
            }
        }
    }
}


 "index_options":"offsets"
 
```

##  第三种 fast vector highlight
1. 对大field而言(大于1mb),性能更好

### 如何设置
```
mapping设置

PUT /my_index
{
    "mappings":{
        "properties":{
            "content":{
                "type":"text",
                "analyzer":"ik_max_word",
                "term_vector":"with_positions_offsets"
            }
        }
    }
}

"term_vector":"with_positions_offsets"
```

# 强制使用某种highlight
对于开启了`term vector`的`field`而言,可以强制使用 `plain highlight`

开启term vector
```
PUT /my_index
{
    "mappings":{
        "properties":{
            "content":{
                "type":"text",
                "analyzer":"ik_max_word",
                "term_vector":"with_positions_offsets"
            }
        }
    }
}

```
使用指定highlight
```
GET /_search
{
    "query":{
        "match":{
            "user":"ki"
        }
    },
    "highlight":{
        "fields":{
            "content":{
                "type":"plain"
            }
        }
    }
}

```

一般情况下 默认的`highlight`就够了  
如果要求高的 可以尝试 `posting highlight`  
如果field特别大,超过了1m ,使用 `fast vector highlight`

# 自定义高亮html标签
默认的是 `<em></em>`标签
添加两个参数: `pre_tags`和`post_tags`
```
GET /_search
{
    "query":{
        "match":{
            "user":"ki"
        }
    },
    "highlight":{
        "pre_tags":["<tag1>"],
        "post_tags":["</tag1>"],
        "fields":{
            "content":{}
        }
    }
}

```

# 高亮文本片段的相关设置
```
GET /_search
{
    "query":{
        "match":{
            "user":"kimchy"
        }
    },
    "highlight":{
        "fields":{
            "content":{
                "fragment_size":150,
                "number_of_fragments":3,
                "no_match_size":150
            }
        }
    }
}

```
1. `fragment_size`  :你一个Field的值,比如有长度是1万,但是你不可能在页面上显示这么多长,  这个可以用来设置显示文本片段的长度 默认100  
注意:必须大于10 不然会提示太小

2. `number_of_fragments`: 你可能高亮的文本有多个高亮片段,你可以指定就显示几个高亮字段 
3. `no_match_size`: 如果说你当前搜索的文本是没有匹配高亮的,你可以手动显示前缀为size大小的字符的文本 返回高亮  
`一般不用这个参数`