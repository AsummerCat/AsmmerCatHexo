---
title: ElasticSearch笔记聚合分析相关(二十四)
date: 2020-08-25 23:22:08
tags: [ElasticSearch笔记]
---

## 核心概念
1. bucket :一个数据分组 类似mysql的`group by`
2. metric: 对一个数据分组执行的统计 `group by`之后的聚合计算

<!--more-->

# histogram 按照区间统计
也是一个bucket分组操作,接收一个field,按照这个field的值在各个范围区间,进行bucket分组操作
```
GET /my_index/_search
{
    "size":0,
    "aggs":{
        "price":{
            "histogram":{
                "field":"price",
                "interval":2000 
                //划分范围  0-2000 ,2000-4000,4000-6000
            },
            "aggs":{
                "revenue":{
                    "sum":{
                        "field":"price"
                    }
                }
            }
        }
    }
}

```



# date_histogram 按照时间区间统计
案例:
```
GET /my_index/_search
{
    "size":0,
    "aggs":{
        "sales":{
            "date_histogram":{
                "field":"sold_date", //分组的时间字段
                "interval":"month",//根据'月'份做区间
                "format":"yyyy-MM-dd",
                "min_doc_count":0, 
                "extended_bounds":{ 
                //在该时间范围内做筛选
                    "min":"2017-01-01",
                    "max":"2017-12-01"
                }
                
            }
        }
    }
}

'min_doc_count' :即使某个区间内都没有数据,这个区间也是要返回的
'extended_bounds': 划分bucket的时候,会限定区间
```

# _global bucket :单个与全部的对比
也就是`group by `后的数据 与`select * `做对比


<font color="red">global</font>:就是global bucket,就是将所有数据纳入聚合的scope,而不管之前的query
## 写法 global:
```
GET /index/_search
{
    "size":0,
    "query":{
        "match":{
            "title":"长虹"
        }
    },
    "aggs":{
        "avg_price":{
            "avg":{
                "field":"price"
            }
        }
    },
    "all":{  //获取所有
        "global":{},
        "aggs":{
            "avg_price":{
                "avg":{
                    "field":"price"
                }
        }
    }
}
```

# fiter+aggs 指定过滤某个内容的平均值
如果放在query那就就是过滤全局的,  
放在aggs里是过滤单个bucket的
```
GET /index/_search
{
    "size":0,
    "query":{
        "trem":{
            "brand":"长虹"
        }
    },
    "aggs":{
        "recent_sales":{
            "filter":{  //先过滤
                "range":{
                    "sold_date":{
                        "get":"now-100d" //100天前
                    }
                }
            }
        },
        "aggs":{
            "avg_price":{
                "avg":{  //后计算
                    "field":"price"
                }
            }
        }
    }
}
```

# _cardinality 去重
`重点:控制内存开销和精准度`

`去重` cartinality   5%的错误率,性能在100ms左右(性能好)  
对每个bucket中指定的field进行去重,取去重后的count,类似于count(distcint)
```
GET /index/_search
{
    "size":0,
    "aggs":{
        "months":{
            "date_histogram":{
                "field":"sold_date",
                "interval":"month"
            },
            "aggs":{
                "distinct_colors":{
                    "cardinality":{ //去重操作 
                        "field":"brand"
                    }
                }
            }
        }
    }
}
```
## 基于去重的优化操作
### 1.precision_threshol提高准确度
`precision_threshold`: 优化准确率和内存开销
```
GET /index/_search
{
            "size":0,
            "aggs":{
                "distinct_colors":{
                    "cardinality":{ //去重操作 
                        "field":"brand",
                        "precision_threshold":100 //优化准确率
                        
                    }
                }
            }
        }
```


在多少个unique value以内,保证100%准确    
如果输入`100` 那么在100个去重结果以内 输出`100%`准确


`precision_threshold * 8byte` 内存消耗  
输入`100`,数百万的unique value,错误率在5%以内  

### 2.接着对HyperLogLog算法进行优化
cardinality底层算法: HyperLogLog算法  
会对所有的uqniue value取hash值,通过hash值近似去求distcint count,误差  
默认情况下,发送一个cardinality请求的时候,会动态地对所有的field value取hash值;  
优化:将取hash值的操作,迁移到建立索引的时候

```
PUT /my_index/_mappings
{
   "properties":{
       "brand":{
           "type":"text",
           "fields":{
               "hash":{  //建立一个hash字段
                   "type":"murmur3" //一种hash算法
               }
           }
       }
   }
}
```
然后直接使用`precision_threshold`就可以了


#  _percentiles 根据计算百分比 进行统计
进行统计50%的人在干嘛  
进行统计40%的人在干嘛  
使用`percentiles算法`
```
GET /index/_search
{
    "size":0,
    "aggs":{
        "latency_percentiles":{
            "percentiles":{
                "field":"进行计算的字段",
                "percents":[ //进行分百分比 1% 5% 10%
                 1,
                 5,
                 10,
                 40,
                 50
                ]
            }
        },
        "latency_avg":{
            "avg":{ //求平均
                "field":"进行计算的字段"
            }
        }
    }
}

```

# percentile_ranks 计算xxx范围内的数据占用的百分比
(常用)
比如:  
场景: 根据平台分类,计算->电视机售价在1000的占比,2000的占比,3000的占比
```
GET /index/_search
{
    "size":0,
    "aggs":{
        "zones":{
            "terms":{
                "field":"province"
            },
            "aggs":{
                "load_times":{
                    "percentile_ranks":{ //计算xxx范围内的数据占用的百分比
                        "field":"latency",
                        "values":[200,1000]  //200以内 和1000以内
                    }
                }
            }
        }
    }
}


```

# 如果不需要 聚合操作可以禁用doc_value减少磁盘空间占用
```
PUT /my_index/_mappings
{
    "properties":{
        "my_field":{
            "type":"keyword",
            "doc_values":false  //禁用doc_value
        }
    }
}

```

# 对buckets(doc)海量数据筛选进行优化
1. 深度优先 <font color="red">(默认)</font>  
意思是: 先构建一个完整的树,再进行筛选数据,
99%的数据都浪费了
```
类似 搜索name=小明 ,age=100 
会直接构建出 整表数据组合的结果(笛卡尔积)再进行筛选 
```
2. 广度优先

```
类似 搜索name=小明 ,age=100 
会直接构建出 name=小明的结果(先过滤出符合结果的数据) 再去过滤age
```
## 广度优先的语法
"collect_mode":"breadth_first" //广度优先
```
GET /my_index/_search
{
   "aggs":{
       "actors":{
           "trems":{
               "field": "actors",
               "size":10,
               "collect_mode":"breadth_first" //广度优先
           },
           "aggs":{
               "costars":{
                   "terms":{
                       "field":"films",
                       "size":5;
                   }
               }
           }
       }
   }
}

原先查询: 先找出actors 和films组成的树 再进行筛选

现在查询: 广度优先,先找出actors的10条数据,在进行查询films的5条数据

```