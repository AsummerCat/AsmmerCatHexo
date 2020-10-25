---
title: ElasticSearch笔记使用geo_point地理位置数据类型(三十三)
date: 2020-08-31 22:01:56
tags: [elasticSearch笔记]
---

## 建立geo_point类型的mapping
```
PUT /my_index
{
    "mappings":{
        "properties":{
            "location":{
                "type":"geo_point"
            }
        }
    }
}

    "type":"geo_point"  //坐标
```

<!--more-->

## 写入geo_point的三种方法
写法一:
```
PUT /my_index/_doc/1
{
    "text":"Geo-point as an object",
    "location":{
        "lat":41.12,
        "lon":-71.34
    }
}

```
写法二:
```
PUT /my_index/_doc/1
{
    "text":"Geo-point as an object",
    "location":"41.12,-71.34"
}
```
写法三:
```
PUT /my_index/_doc/1
{
    "text":"Geo-point as an object",
    "location":[41.12,-71.34]
}
```

# 根据地理位置查询
查询某个矩形的地理范围内的坐标点

## <font color="red">geo_bounding_box --- 建立矩形</font>

```
GET /my_index/_search
{
  "query": {
    "geo_bounding_box":{
      "location":{
        "top_left":{
          "lat":42,
          "lon":-72
        },
        "bottom_right":{
          "lat":40,
          "lon":-74
        }
      }
    }
  }
}
会根据 'left 左上角的点' 和'right 右下角的点' 的两个点,  
 来划出一个矩形 查找出该范围内的点
```

## <font color="red">geo_polygon --- 建立多边形 可描绘多个点</font>

```
GET /my_index/_search
{
    "query":{
        "bool":{
            "must":{
                "match_all":{}
            },
            "filter":{
                "geo_polygon":{
                    "location":{
                        "points":[
                        {"lat":40.73,"lon":-74.1},
                        {"lat":40.01,"lon":-71.12},
                        {"lat":50.56,"lon":-90.58}
                        ]
                    }
                }
            }
        }
    }
}


```

## <font color="red">geo_distance--- 根据距离来获取坐标</font>

```

GET /my_index/_search
{
  "query":{
    "bool":{
      "must":[
        {
          "match_all": {}
        }
        ],
        "filter":{
          "geo_distance":{
            "distance": "100km",
            "location": {
              "lat": 40.73,
              "lon": -74.1
            }
          }
        }
    }
  }
}

能搜索出来距离location 100km范围内的坐标
```

## <font color="red">geo_distance--- 查出每个距离内有多少个坐标</font>

```
GET /my_index/_search
{
  "size": 0,
  "aggs":{
    "rings_around_amsterdam":{
      "get_distance":{
        "field":"location",
        "origin":"40,-70",
        "unit":"m",
        "distance_type":"plane",
        "ranges":[
          {"to":100},
          {"from":100,"to":300},
          {"from":300}
          ]
      }
    }
  }
}

在'origin'
取出三个范围内分别有多少个坐标点 , 0-100   ,100-300  ,300以上

'unit'的单位: 'm,mi,in,yd,km,cm,mm'

'distance_type'的类型: '查找范围点的方法' 
      1. sloppy_arc  默认的
      2. arc   (最精准 速度慢)
      3. plane (不精准 速度快)
```

