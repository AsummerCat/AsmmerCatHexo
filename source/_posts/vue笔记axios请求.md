---
title: vue笔记axios请求
date: 2020-06-19 17:25:39
tags: [vue]
---

# vue笔记axios请求
用来发送ajax请求

## axios安装
```
cnpm install --save axios vue-axios

cnpm install
```
## 在main.js导入
```
import Vue from 'vue'
import axios from 'axios'
import VueAxios from 'vue-axios'

Vue.use(VueAxios,axios)

```
<!--more-->

## 发送ajax请求
```
this.axios({
    method:'get',
    url:'www.baidu.com',
    data:{}
}).then(function(res){
console.log(res.data);
})


Axios.get('demo/url', {
    params: {
        id: 123,
        name: 'Henry',
        sex: 1,
        phone: 13333333
    }
})

```

## 服务端解决跨域问题
```
<mvc:cors>
  <mvc:mapping path="/**"
        allowed-origins="*"
        allowed-methods="POST,GET,OPTIONS,DELETE,PUT,PATCH"
        allowed-headers="Content-Type,Access-Contrl-Allow-Headers,Authorization,X-Requested-With"
        allowed-credentials="true"/>
</mvc:cors>

在spring-mvc.xml
加入跨域的配置
allowed-origins:表示允许访问源的域名
```

## 解决axios无法传递data的问题 导入axios内置的qs包
```java
默认的请求头格式是:
Content-Type:applcation/json;charset=UTF-8

如果后台接收的类型是:
Content-Type:applcation/x-www-from-urlencoded

可以使用内置qs包 转换格式传入

比如:
import QS from 'qs'
    
    let _les=this;
 this.axios({
     method:'POST',
     url: 'http://baidu.com',
     transfromRequest:[function(data){ //这里就是Qs进行转换
          return Qs.stringify(data)
     }],
     data:{
         email:_les.email
     }
 }).then(function(res){
console.log(res.data);
})

 

```

## 使用   提供Mock假数据
主要是为了 发送请求的时候 不用再等待后台完成接口
前台可以先做测试

```
有很多第三方的 百度一下使用
比如 easy mock 在线创建接口 来替换指定后端接口地址 来调试
```