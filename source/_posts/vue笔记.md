---
title: vue笔记
date: 2020-06-19 17:24:55
tags: [vue]
---

# vue笔记

## ref的使用
类似id的效果
```
页面:
<div ref="test"></div>

vue获取:
this.$refs.test.xxxxx;
```

## template的作用
template的内容会直接渲染在 对应的el标签中
```
页面:
<div id="app1"></div>


new vue({
    el: "#app1",
    template: '<h1>hello</h1>'
})

```
<!--more-->

## $mound的作用
动态绑定页面元素id

```
var v1=new vue({
    template: '<h1>hello</h1>'
})

# 绑定
v1.$mound('#app1');

```

## vue组件 component 

Vue.component("组件名",{
vue对象
})
```
全局创建并注册组件方式:
Vue.component("hello",{
template: "<div>hello word </div>"
})


全局注册组件方式 (多个):
//导入:
import hello from './com/hello.vue'
import hello1 from './com/hello1.vue'
import hello2 from './com/hello2.vue'
//注册1
Vue.components({
"hello":组件名称,
"hello1":组件名称1,
hello2":组件名称2,
})
//注册2 (单个)
Vue.component("hello",组件名称);
Vue.component("hello1",组件名称1);


本地注册组件:
# 1.创建模板
var emp:{
   template: "<div>hello word </div>" 
}
# 2.注册到本地 
new Vue({
    el:"#app1",
    ## 组件组件到本地
    components:{
       '组件名称': emp
    }
})



页面使用:
<hello></hello>

```