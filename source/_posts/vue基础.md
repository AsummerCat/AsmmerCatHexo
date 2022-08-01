---
title: vue基础
date: 2020-06-02 19:47:36
tags: [vue]
---

# 基础语法
## 常用使用

elemenUI 做页面
node.js 
vue-cli脚手架
vur-router路由

## 常见属性,方法

```
v-if
v-else-if
v-else
v-for
v-on 绑定事件,简写@
v-model数据双向绑定
v-bind 给组件绑定参数 ,简写:  v-bind:qin="数据"

```

<!--more-->

## 定义组件

```
方式一:
页面直接创建:
Vue.component("组件名称",{
    <!--接收参数 不能大写-->
    props:['qin'],
    <!--模板内容-->
    template:  '<li>{{qin}}</li>'
    
});

方式二:
使用 .Vue文件


页面使用:
 <!--组件: 传递给组件中的值: props-->
<组件名称 v-bind:qin="数据"></组件名称>

```

## 导入组件
```
在HelloWorld.vue中 写入:
基本上每个.vue文件都需要这个
显示的暴露出接口
export default{
     name: 'HelloWorld',
     data:{
         return {
         msg:"测试数据1";
         }
     }
 }

import 组件名称 from '组件位置'

比如: import HelloWorld from './HelloWorld'

使用: 
在 App.vue文件中
 export default{
     name: 'App',
     components:{
     <!--加载了HelloWord的组件
         'HelloWorld'
     }
 }

```

## 定义数据
```
var vm =new Vue({
    el: "#+标签ID", 
    data:{
    <!--定义数据-->
        items: ["java","linux"]
    }
    
})

```

## Axios异步通信 ->发送请求
idea 修改为es6.0 `javaScript ->version`
```
安装:
npm install axios
npm install --save axios vue-axios
```

```
导入: 
import Vue from 'vue'
import axios from 'axios';
import VueAxios from 'vue-axios'

Vue.use(VueAxios,axios)
```

```
使用:
Vue.axios.get(api).then((response)->{
 console.log(response.data)
})
```

## 钩子函数
```
var vm =new Vue({
    el: "#+标签ID", 
    <!--数据-->
    data(){
       return:{
           //请求的返回参数,必须和json字符串一样
           info:{
               name: null,
               name: null
           }
       }
    },
    <!--钩子函数 一般用于ajax请求获取数据进行数据初始化-->
    <!-- 整个实例只执行一次 mounted-->
        mounted(){  
           <!--链式编程 发送请求接收数据返回-->
            axios.get('../data.json').then(response=>
            (this.info=response.data)
            );
        }
})
```

## 解决数据加载前出模板内容
```
<style>
[v-clock]{
    display: none;
}
</style>



标签上写入: <div id="xxx" v-clock></div>
```

## 计算属性

```
var vm =new Vue({
    el: "#app",
    data: {
        message: "hello,word"
    },
    //定义方法
    methods:{
         currentTime1: function(){
             return Date.now(); //返回当前时间戳
         }
    },
    //定义计算 属性
    computed:{
       currentTime1: function(){
             return Date.now(); //返回当前时间戳
         } 
    }
})


页面调用: 

<div id="app">
  methods调用方法+(): {{currentTime1()}}
  computed调用属性 : {{currentTime1}}
</div>

```

## 自定义事件分发

 在vue的标签中添加
```
 
 v-on:自定义方法="引用方法(参数)"
 
 比如:
 
 <div id="app">
  <div v-for="(item,index) in todoItems"
   v-bind:index="index" v-on:remove="removeItem(index)" > 
    </div>
 </div>
 
 nVue.component("todo",{
   methods:{
       remove: function(index){
           //this.%emit 自定义事件分发 (获取父页面的方法)
           this.$emit('remove',index);
       }
   }
 })
 这样可以引用其他new vue的方法
```
 父传子: props
 子传父: $emit


 ## 引入js

 ### 使用严格编写模式
 在js第一行 写入
```
 `use strict`
```
 来保证编译期的语法完整


 # vue-cli脚手架

 ## 安装node.js
```
 1. node+npm下载
 2.查看版本 : 
 node -v   npm -v
 3. 全局安装  : 
 npm install cnpm -g
 4. 使用加速器下载镜像:
 npm install --registry=https://registry.npm.taobao.org
```
 ## 安装vue-cli
```
 cnpm install vue-cli -g
 
 --查看可以基于哪些模板创建vue项目 
 vue list
```
 ## 创建一个vue项目
 使用webpack创建
```
 vue init webpack 项目名称
```
 ## 初始化并运行
```
 cd xxx项目目录
 npm install
 npm run dev
```

 # webpack

 ## 安装webpack
```
 npm install webpack -g
 npm install webpack-cli -g
 
 查看版本 webpack-cli -v
```
 ## 项目结构
 创建webpack.config.js配置文件
```
-  entry: 入口文件,指定webpack用哪个文件作为项目的入口
-  output: 输出,指定webpack吧处理完成的文件放置到指定路径
-  module: 模块,用于处理各种类型的文件
-  plugins: 插件,如:热更新,代码重用等
-  resolve: 设置路径指向
-  watch: 监听,用于设置文件改动后直接打包
 
 
```

## 安装vue-router

```
npm install vue-router --save-dev
```
### 使用:
```
import Vue from 'vue'
import VueRouter from 'vue-fouter'
Vue.use(VueRouter);
```
