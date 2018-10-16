---
title: easyui动态添加空间无法渲染$.parser.parse无效
date: 2018-10-15 10:16:56
tags: 前端
---

# easyui 动态添加空间无法渲染 $.parser.parse()无效

## 解决方案:

`$.parser.parse($('#judge_logic_').parent());  `

---
---

<!--more-->

动态添加easyui控件`<input class="easyui-combobox" >` 这样是无效的，因为easyui没有实时监控，所以必须动态渲染

```
$.parser.parse();
$.parser.parse(context) 
```
 
//context  为待查找的 DOM 元素集、文档或 jQuery 对象,为空时默认为整个文档  
//渲染对象为： `class="easyui-pluginName"`的元素  

```
注意  如果想通过id 获取 jquery对象来获取的话必须       
 $.parser.parse($('#judge_logic_').parent());  
 后面必须有一个   .parent()  否则无效

```
 
像下面代码去手工解析的话是得不到你想要的结果的：

`$.parser.parse($('#tt'));  `

道理很简单，parser只渲染tt的子孙元素，并不包括tt自身，而它的子孙元素并不包含任何Easyui支持的控件class，所以这个地方就得不到你想要的手风琴效果了，应该这样写：

`$.parser.parse($('#tt').parent());`


渲染tt的父节点的所有子孙元素就可以了，个人觉得通过jQuery的parent()方法是最安全不过的了，不管你的javascript输出了什么DOM，直接渲染其父节点就可以保证页面能被正确解析
