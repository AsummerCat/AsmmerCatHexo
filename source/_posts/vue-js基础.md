---
title: vue.js基础
date: 2018-11-07 17:08:56
tags: vue
---

# vue.js
渐进式
JavaScript 框架  

[官网](https://cn.vuejs.org/)
[文档](https://cn.vuejs.org/v2/guide/instance.html)

# vue语法

每个 Vue 应用都需要通过实例化 Vue 来实现。

语法格式如下：

```
var vm = new Vue({
  // 选项
})
```

<!--more-->

# 函数定义

```
接下来我们看看如何定义数据对象。

data 用于定义属性，实例中有三个属性分别为：site、url、alexa。

methods 用于定义的函数，可以通过 return 来返回函数值。

{{ }} 用于输出对象属性和函数返回值。
```

## 例子

```
<!DOCTYPE html>
<html>
<head>
	<meta charset="utf-8">
	<title>Vue 测试实例 - 菜鸟教程(runoob.com)</title>
	<script src="https://cdn.staticfile.org/vue/2.4.2/vue.min.js"></script>
</head>
<body>
   <div id="vue_det">
    <h1>site : {{site}}</h1>
    <h1>url : {{url}}</h1>
    <h1>{{details()}}</h1>
</div>
<script type="text/javascript">
	var data={
            site: "菜鸟教程",
            url: "www.runoob.com",
            alexa: "10000"
        }
    var vm = new Vue({
        el: '#vue_det',
        data: data,
        methods: {
            details: function() {
                return  this.site + " - 学的不仅是技术，更是梦想！";
            }
        }
    })
    //改变值
	data.url="hehehe"
	vm.url="111"
	</script>
</body>
</html>

```
<font color="red">  
因为定义了 var vm =new Vue{}  
name  vm.a==data.a  
所以改变vm.XX等同于data.xx 效果相等
如果改变data的属性 页面上的值 也会对应的进行改变</font>  

 需要注意的是: 只有实例创建的时候 data中存在的属性才是响应式的

---

# Object.freeze() 阻止修改现有属性 ==无法响应式
## 例子

```
var data = {
  foo: 'bar'
}

//注意如果已经生成了实例 这个也就没有用了
Object.freeze(data)

new Vue({
  el: '#app',
  data: data
})

data.url="hehehe"

这样的话 foo 还是等于 bar 而不会被修改为hehehehe
```

# $watch()  这里可以在摸个属性值被修改时候触发(实例方法)

## 例子

```
// 我们的数据对象
	var data = { site: "菜鸟教程", url: "www.runoob.com", alexa: 10000}
	
	var vm = new Vue({
		el: '#vue_det',
		data: data
	})
	
	// 
vm.$watch('url', function (newValue, oldValue) {
  // 这个回调将在 `vm.a` 改变后调用
	alert("newValue"+newValue+"oldValue"+oldValue)
})
	data.url=1


当data.url 改变为1的时候触发这个方法
```

# 模板语法

##  {{ msg }}
文本 最常用的 直接就

### v-once 指令
如果要一次性插值 之后不在改变 可以这么写  
通过使用 v-once 指令，你也能执行一次性地插值，当数据改变时，插值处的内容不会更新。但请留心这会影响到该节点上的其它数据绑定

```
<span v-once >Message: {{ msg }}</span>
```

### v-html="" 指令
这里是将html直接输入到页面 其他只是输出文本

```
<p>Using v-html directive: <span v-html="rawHtml"></span></p>
```
这个 span 的内容将会被替换成为属性值 rawHtml

##  v-bind 绑定 
可以用来绑定参数 或者动态来绑定 属性  
例如

```
<h1 v-html="url" v-bind:hidden="test">url : {{url}}</h1>

<div class="static" v-bind:class="{ active: isActive, 'text-danger': hasError }">
</div>


<script type="text/javascript">
	// 我们的数据对象
	var data = { site: "菜鸟教程", url: "www.runoob.com", alexa: 10000,test:true}
	
	var vm = new Vue({
		el: '#vue_det',
		data: data
	})
	data.url="<font color='red'>1</font>"

	</script>
然后我们在绑定的属性中 修改 test  

test: true 显示   test:false 隐藏
```

##使用 JavaScript 表达式 

例子

```
可以直接这么写

<h1 v-html="url+'111'" >url : {{url}}</h1>
{{ number + 1 }}

{{ ok ? 'YES' : 'NO' }}

{{ message.split('').reverse().join('') }}


tips: 需要注意的是 (不生效)
<!--流控制也不会生效，请使用三元表达式 --> 
{{ if (ok) { return message } }}

```
输出为 <font color="red">1</font>111


## v-if  根据表达式 seen 的值的真假来插入/移除元素

例子

```
<p v-if="seen">现在你看到我了</p>
```
##  v-on 指令，它用于监听 DOM 事件
例子:

```
<a v-on:click="doSomething">...</a>

这里可以写js方法 也可以是哟很new vue下的 methods中的方法  
```

# 缩写
## v-bind 缩写

```
<!-- 完整语法 -->
<a v-bind:href="url">...</a>


<!-- 缩写 -->
<a :href="url">...</a>
```

## v-on 缩写

```
<!-- 完整语法 -->
<a v-on:click="doSomething">...</a>

<!-- 缩写 -->
<a @click="doSomething">...</a>
```

---


# v-if 条件渲染
在字符串模板中，比如 Handlebars，我们得像这样写一个条件块：

```
<!-- Handlebars 模板 -->
{{#if ok}}
  <h1>Yes</h1>
{{/if}}
```

在 Vue 中，我们使用 v-if 指令实现同样的功能：
```
<h1 v-if="ok">Yes</h1>
```

也可以用 v-else 添加一个“else 块”：

```
<h1 v-if="ok">Yes</h1>
<h1 v-else>No</h1>
```

`v-else-if`

类似于 v-else，v-else-if 也必须紧跟在带 v-if 或者 v-else-if 的元素之后。

---

# v-show 显示隐藏 会保留在dom中

```
v-if 是不渲染 
v-show只是显示/隐藏

另一个用于根据条件展示元素的选项是 v-show 指令。用法大致一样：

<h1 v-show="ok">Hello!</h1>
```

tips: 注意，v-show 不支持 <template> 元素，也不支持 v-else。

---


#  用 key 管理可复用的元素

Vue 会尽可能高效地渲染元素，通常会复用已有元素而不是从头开始渲染。这么做除了使 Vue 变得非常快之外，还有其它一些好处。例如，如果你允许用户在不同的登录方式之间切换：

例子:

```
<template v-if="loginType === 'username'">
  <label>Username</label>
  <input placeholder="Enter your username">
</template>
<template v-else>
  <label>Email</label>
  <input placeholder="Enter your email address">
</template>


这样切换的话 内容还是 input 内容还是不会消失的  需要key 让他不复用


<template v-if="loginType === 'username'">
  <label>Username</label>
  <input placeholder="Enter your username" key="username-input">
</template>
<template v-else>
  <label>Email</label>
  <input placeholder="Enter your email address" key="email-input">
</template>
这样每次都会重新渲染
```

---

# v-for 循环语句


我们用 v-for 指令根据一组数组的选项列表进行渲染。v-for 指令需要使用 item in items 形式的特殊语法，items 是源数据数组并且 item 是数组元素迭代的别名。

```script
<ul id="example-1">
  <li v-for="item in items">
    {{ item.message }}
  </li>
</ul>


var example1 = new Vue({
  el: '#example-1',
  data: {
    items: [
      { message: 'Foo' },
      { message: 'Bar' }
    ]
  }
})

在 v-for 块中，我们拥有对父作用域属性的完全访问权限。v-for 还支持一个可选的第二个参数为当前项的索引。

<ul id="example-2">
  <li v-for="(item, index) in items">
    {{ parentMessage }} - {{ index }} - {{ item.message }}
  </li>
</ul>

你也可以用 of 替代 in 作为分隔符，因为它是最接近 JavaScript 迭代器的语法：

<div v-for="item of items"></div>

```

##  一个对象的  v-for

```script
你也可以用 v-for 通过一个对象的属性来迭代。

<ul id="v-for-object" class="demo">
  <li v-for="value in object">
    {{ value }}
  </li>
</ul>
new Vue({
  el: '#v-for-object',
  data: {
    object: {
      firstName: 'John',
      lastName: 'Doe',
      age: 30
    }
  }
})
```

 也可以多一个参数作为key 

```script
<div v-for="(value, key) in object">
  {{ key }}: {{ value }}
</div>
```

第三个参数为索引：

```
<div v-for="(value, key, index) in object">
  {{ index }}. {{ key }}: {{ value }}
</div>
```

---



