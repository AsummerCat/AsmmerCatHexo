---
title: ElasticSearch笔记使用Search_template将搜索模板化(三十)
date: 2020-08-31 21:59:37
tags: [ElasticSearch笔记]
---

## 创建搜索模板
```
GET /xx/_search/template
{
   "source":{
        "query":{
            "match":{
                "{{field}}":"{{value}}"
            }
        }
    },
    "params":{
        "field":"title",
        "value":"博客"
    }
}

```
<!--more-->

## 以json的形式书写模板
不单单可以传递 占位符 还可以传入一段内容
```
GET /xx/_search/template
{
  "source":"{\"query\":{\"match\":{{#toJson}}matchCondition{{/toJson}}}}",
      "params":{
        "matchCondition":{
          "title":"博客"
        }
    }
}

```

## 以join的形式拼接内容 类似string.join
```
GET /xx/_search/template
{
  "source": {
    "query": {
      "match": {
        "title": "{{#join delimiter=','}}titles{{/join delimiter=','}}"
      }
    }
  },
  "params": {
    "titles": [
      "博客",
      "网站"
    ]
  }
}

```

## 复杂的模板
```
{
  "source":{
    "query":{
      "bool":{
        "must":{
          "line":"{{text}}"
        }
      },
      "filter":{
        {{#line_no}}
        "range":{
          "line_no":{
            {{#start}}
              "gte":"{{start}}"
              {{#end}},{{/end}}
            {{/start}}  
            {{#end}}
              "lte":"{{end}}"
            {{/end}} 
          }
        }
        {{/line_no}}
      }
    }
  }
}
```

# 保存模板
es的`config/script`目录下,预先保存这个复杂的模板,后缀名是 `.mustache`
# 模板的内容就是
```
{
  "source":{
    "query":{
      "bool":{
        "must":{
          "line":"{{text}}"
        }
      },
      "filter":{
        {{#line_no}}    //line_no必须是有值的才走该逻辑
        "range":{
          "line_no":{
            {{#start}}
              "gte":"{{start}}"
              {{#end}},{{/end}}
            {{/start}}  
            {{#end}}
              "lte":"{{end}}"
            {{/end}} 
          }
        }
        {{/line_no}}
      }
    }
  }
}
```

# 使用脚本
```
GET /my_index/my_type/_search/template
{
    "file":"conditional",
    "params": {
       "title":"博客",
       "line_no":true,
       "start":1,
       "end":10
  }
}

```
带上`file=模板名称`