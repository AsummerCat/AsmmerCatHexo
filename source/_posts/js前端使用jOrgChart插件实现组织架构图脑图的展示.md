---
title: js前端使用jOrgChart插件实现组织架构图脑图的展示
date: 2020-06-04 14:32:19
tags: [前端]
---

#  js前端使用jOrgChart插件实现组织架构图脑图的展示

## 文档地址

[文档地址](https://github.com/wesnolte/jOrgChart)

```
需要引入的js插件和css文件：

　　①jquery.jOrgChart.css

　　②jquery.min.js

　　③jquery.jOrgChart.js

使用jOrgChart插件，根据返回的数据将其子节点加入到相应的<li></li>中。
```

<!--more-->

我们的数据表应该要有 id（节点），pid（父节点的id），name的字段，

### 数据定义

注意：

　　后台返回的数据格式必须如下，其中id，pid字段为必须要有。childrens字段也为必须的，不过字段名可以自己定义，name字段是根据自己业务需求的字段，在这里以name字段作为要显示的文本内容：

```
{
  "data": [{
    "id": 1,
    "name": "企业主体信用得分",
    "pid": null,
    "childrens": [
      {
        "id": 2,
        "name": "企业素质",
        "pid": 1,
        "childrens": [
          {
            "id": 5,
            "name": "基本信息",
            "pid": 2,
            "childrens": [
              {
                "id": 10,
                "name": "企业主体信息识别",
                "pid": 5,
                "childrens": [
                ]
              },
              {
                "id": 11,
                "name": "企业持续注册时间",
                "pid": 5,
                "childrens": [
                ]
              },
              {
                "id": 12,
                "name": "注册资本",
                "pid": 5,
                "childrens": [
                ]
              }
            ]
          },
          {
            "id": 6,
            "name": "管理认证",
            "pid": 2,
            "childrens": [
              {
                "id": 13,
                "name": "国际性管理认证",
                "pid": 6,
                "childrens": [
                ]
              }
            ]
          }
        ]
      },
      {
        "id": 3,
        "name": "履约记录",
        "pid": 1,
        "childrens": [
          {
            "id": 7,
            "name": "税务执行情况",
            "pid": 3,
            "childrens": [
              {
                "id": 14,
                "name": "是否按时缴纳税款",
                "pid": 7,
                "childrens": [
                ]
              }
            ]
          },
          {
            "id": 8,
            "name": "网贷情况",
            "pid": 3,
            "childrens": [
              {
                "id": 15,
                "name": "网贷逾期",
                "pid": 8,
                "childrens": [
                ]
              }
            ]
          }
        ]
      },
      {
        "id": 4,
        "name": "公共监督",
        "pid": 1,
        "childrens": [
          {
            "id": 9,
            "name": "行政处罚",
            "pid": 4,
            "childrens": [
              {
                "id": 16,
                "name": "处罚信息",
                "pid": 9,
                "childrens": [
                ]
              }
            ]
          }
        ]
      }
    ]
  }
]
}
```

### 前端页面处理

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>jOrgChart异步加载</title>
    <link rel="stylesheet" href='jquery.jOrgChart.css'/>
    <script type='text/javascript' src='jquery.min.js'></script>
    <script type='text/javascript' src='jquery.jOrgChart.js'></script>
    <style>
        a {
            text-decoration: none;
            color: #fff;
            font-size: 12px;
        }
        .jOrgChart .node {
            width: 120px;
            height: 50px;
            line-height: 50px;
            border-radius: 4px;
            margin: 0 8px;
        }
    </style>
</head>
<body>
<!--显示组织架构图-->
<div id='jOrgChart'></div>


<script type='text/javascript'>
    $(function(){
        //数据返回
        $.ajax({
            url: "test.json",
            type: 'GET',
            dataType: 'JSON',
            data: {action: 'org_select'},
            success: function(result){
                var showlist = $("<ul id='org' style='display:none'></ul>");
                showall(result.data, showlist);
                $("#jOrgChart").append(showlist);
                $("#org").jOrgChart( {
                    chartElement : '#jOrgChart',//指定在某个dom生成jorgchart
                    dragAndDrop : false //设置是否可拖动
                });

            }
        });
    });

    function showall(menu_list, parent) {
        $.each(menu_list, function(index, val) {
            if(val.childrens.length > 0){

                var li = $("<li></li>");
                li.append("<a href='javascript:void(0)' onclick=getOrgId("+val.id+");>"+val.name+"</a>").append("<ul></ul>").appendTo(parent);
                //递归显示
                showall(val.childrens, $(li).children().eq(1));
            }else{
                $("<li></li>").append("<a href='javascript:void(0)' onclick=getOrgId("+val.id+");>"+val.name+"</a>").appendTo(parent);
            }
        });

    }

</script>
</body>
</html>
```

### 效果图

注意：由于数据是由ajax异步请求获取到的，所以直接双击html文件打开是不行的，需要在服务器环境下运行。

![](/img/2020-06-03/jq.png)