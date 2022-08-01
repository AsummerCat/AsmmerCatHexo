---
title: EasyUI的Tree结构
date: 2018-09-27 14:32:31
tags: [前端]
---
 ## 1.引入JS

 **[文档地址](http://www.jeasyui.net/plugins/185.html)**
 
 <!--more-->

```
<script src="${ctx}/static/libs/js/plugins/easyui/jquery.easyui.min.js"></script>

<ul id="tree"></ul>  //空树
```

---

## 2.结构

```script
[{
    "id":1,
    "text":"Folder1",
    "iconCls":"icon-save",
    "children":[{
       "text":"File1",
       "checked":true
     },{
       "text":"Books",
       "state":"open",
       "attributes":{
          "url":"/demo/book/abc",
          "price":100
       },
       "children":[{
          "text":"PhotoShop",
          "checked":true
          },{
          "id": 8,
          "text":"Sub Bookds",
          "state":"closed"
        }]
     }]
},{
    "text":"Languages",
    "state":"closed",
    "children":[{
       "text":"Java"
    },{
       "text":"C#"
     }]
}]
```

```
  list<map>    map属性: id ,title ,是否有下级   
  <list>children: list   
    "state":"closed", 表示缩略 默认open
```

---

## 2.基本属性

```
名称              类型             描述	                   默认值
url              string	    获取远程数据的 URL 。              null
method	         string	  检索数据的 http 方法（method）       post
animate         boolean	 定义当节点展开折叠时是否显示动画效果。   false
checkbox        boolean	 定义是否在每个节点前边显示复选框。      false
cascadeCheck	boolean	 定义是否级联检查。	                  true
onlyLeafCheck	boolean	 定义是否只在叶节点前显示复选框。       false
lines           boolean	定义是否显示树线条。                  false
dnd	            boolean	定义是否启用拖放。	                 false

```

## 3.异步树

```
  function InitTreeData() {
            $('#tree').tree({
            //url 这个表示唯一的查询入口
               url: ctx + '/city/listAllAreaData', 
               //打开复选框
               checkbox: true, 
                cache: false,
                onlyLeafCheck:true
            });
        }
```

---

## 4.后台写法

```
 public List listAysnAreaData(String id){
        List<Map<String,Object>> list=new ArrayList<>();
            //首次进入查询:
            String sql = "select id,lt_title from t_area where lt_level=0 ";
            List<Map<String,Object>> reqContinent= simpleJdbcTemplate.queryForList(sql);
            for (Map<String,Object> continentInfo:reqContinent) {
                Map<String,Object> continent=Maps.newHashMapWithExpectedSize(3);
                Long continentId=MapUtils.getLong(continentInfo,"id");
                String continentName=continentInfo.get("lt_title").toString();
                continent.put("id",continentId);
                continent.put("text",continentName);
                continent.put("state","closed");
                //如果有下一级别的话
                // continent.put("children",list集合);
                list.add(continent);
            }
        }
        return list;
    }

```

## 获取属性

```
var node = $('#tree').tree('getSelected');
        var data = {};

        if (node.id == undefined) {
            $('#tree').tree('reload');
        }

        var obj = $('#tree').tree('getChildren', node.target);
        if ($(obj).size() > 0) {
            swal({
                title : "保存失败",
                text : '该节点下存在子节点，不能删除！'
            });
            return;
        }

```