---
title: '模板引擎:Handlebars'
date: 2018-11-09 10:12:02
tags:  [前端模板引擎]
---

# Handlebars
## 全球最受欢迎的模板引擎
Handlebars是全球使用率最高的模板引擎,所以当之无愧是全球最受欢迎的模板引擎.Handlebars在许多前端框架中都被引入,比如在MUI和AmazeUI等框架,都推荐使用Handlebars.以AmazeUI为例,AmazeUI的文档中专门为Web组件提供了其Handlebars的编译模板

<!--more-->

---

## 如何使用Handlebars
下载Handlebars 
 
* 通过Handlebars官网下载: http://handlebarsjs.com./installation.html  
* 通过npm下载: npm install --save handlebars  
* 通过bower下载: bower install --save handlebars  
* 通过Github下载: https://github.com/daaain/Handlebars.git  
* 通过CDN引入:https://cdnjs.com/libraries/handlebars.js  


## 基本导入

```
Amaze UI 提供的开发模板中，包含一个 widget.html 文件，里面展示了 Widget 在纯浏览器环境中的使用。

要点如下：

1.引入 Handlebars 模板 handlebars.min.js；
2.引入 Amaze UI Widget helper amui.widget.helper.js；
3.根据需求编写模板 <script type="text/x-handlebars-template" id="amz-tpl">{{>slider slider}}</script>；
4.传入数据，编译模板并插入页面中。

$(function() {
//获取模板位置
  var $tpl = $('#amz-tpl');
  //获取模板内容
  var source = $tpl.text();
  var template = Handlebars.compile(source);
  var data = {};
  var html = template(data);

  $tpl.before(html);
});

```

## 引入Handlebars
通过`<script>`标签引入即可,和引入jQuery库类似:

```
<script src="./js/handlebars-1.0.0.beta.6.js"></script>
```

---

## 创建模板

* 步骤一: 通过一个`<script>`将需要的模板包裹起来  
* 步骤二: 在`<script>`标签中填入type和id  
type类型可以是除text/javascript以外的任何MIME类型,但推荐使用`type="text/template"`,更加语义化
id是在后面进行编译的时候所使用,让其编译的代码找到该模板.  
* 步骤三: 在`<script>`标签中插入我们需要的html代码,根据后台给我们的接口文档,修改其需要动态获取的内容

```
<script type="text/template" id="myTemplate">
    <div class="demo">
        <h1>{{name}}</h1>
        <p>{{content}}</p>
    </div>
</script>
```

## 在JS代码中编译模板

```
//用jQuery获取模板
var tpl   =  $("#myTemplate").html();
//预编译模板
var template = Handlebars.compile(tpl);
//匹配json内容
var html = template(data);
//输入模板
$('#box').html(html);

```

以上述代码为例进行解释:

* 步骤一: 获取模板的内容放入到tpl中,这里`$("#myTemplate")`中填入的内容为你在上一步创建模板中所用的`id`.<font color="red">tips: 这里我使用的`jQuery`的选择器获取,当然,你可以使用原生javascript的DOM选择器获取,例如:`docuemnt.getElementById('myTemplate')`和`document.querySelector('#myTemplate')`</font>


* 步骤二: 使用`Handlebars.compile()`方法进行预编译,该方法传入的参数即为获取到的模板  
* 步骤三: 使用`template()`方法进行编译后得到拼接好的字符串,该方法传入的参数即为上一步预编译的模板.  
* 步骤四: 将编译好的字符串插入到你所希望插入到的`html`文档中的位置,这里使用的是`jQuery`给我们提供的`html()`方法.同样,你也可以使用原生的`innerHTML`

---

## 在JS代码中编译模板

```
//定义getList()函数来发送Ajax请求,传递的参数为后台给的接口文档中定义的参数
function getList(categoryId,pageid){
    //调用jQuery的Ajax()方法来发送Ajax请求
    $.ajax({
        type:'get',
        url:'http://182.254.146.100:3000/api/getproductlist',
        data:{
            pageid:pageid||1,
            categoryid:categoryId
        },
        success:function(data){
            //用zepto获取模板
            var tpl   =  $("#product-list-tepl").html();
            //预编译模板
            var template = Handlebars.compile(tpl);
            //匹配json内容
            var html = template(data);
            //插入模板,到ul中
            $('.product-list ul').html(html);
        }
    })
}

```

