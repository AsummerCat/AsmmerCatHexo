---
title: ElasticSearch笔记搜索的时候自动纠错(二十二)
date: 2020-08-23 15:40:57
tags: [ElasticSearch笔记]
---

# ElasticSearch笔记解决搜索时候填写错误(二十二)

## 使用fuzzy搜索技术
fuzzy搜索技术 -> 自动将拼写错误的搜索文本,进行纠正,纠正以后尝试匹配索引中的数据.
<!--more-->

例如:
```
GET /my_index/_search
{
    "query":{
        "fuzzy":{
            "text":{
                "value":"surprise",
                "fuzziness":2 //也可填入 "auto" 
            }
        }
    }
}

原本我们需要搜索的文本是'superize',
结果输入错误变为'superise', 
fuzzy搜索以后会自动化将 s->z
然后去跟文本进行匹配

'fuzziness' :你的搜索文本最多可以纠正几个字母
默认不设置为2
```