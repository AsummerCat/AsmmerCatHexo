---
title: vue笔记路由相关知识
date: 2020-06-24 22:31:17
tags: [vue]
---

# vue笔记路由相关知识
组件间的跳转

## 安装路由模块
```
npm install vue-router -s
```

## 创建静态路由表
在src下创建router.js
```
import Home form '@/component/Home.vue'
import user form '@/component/user.vue'

//映射关系
export const routes=[
{path:'/',component:home},
{path:'/user',component:user}
]
```
<!--more-->
## 导入路由模块并使用
### 第一种使用静态路由表
在main.js导入路由模块并使用
```
import Vue from 'vue'
import App from './App.vue'
import VueRouter from 'vue-router'// 1.引入路由模块
import {routes} from './routes' //2.引入静态路由表

Vue.use(VueRouter);//3.使用路由模块

//4.创建一个VueRouter模块的实例
const router=new VueRouter({
    routes:routes
})


new Vue({
 el: '#app',
 router, //把router实例放入vue实例中
 render: h => h(App)
})
```
### 第二种 直接创建vueRouter路由
```
export default new Router({
routes:[
    {
    path:'/', //路由路径
    component:home,
    name: 'home'    //路由名称 
    },
    {
    path:'/user',
    component:user
    name: 'user'   
    }
]
})
```


## 路由进行跳转的两种方式
#### 第一种 router-link
```
# 路由链接 跳转路由
<router-link to="/home"></router>

# 路由显示的内容在这里展示
<router-view></router-view>

```
#### 第二种 通过js跳转
设置一个方法
执行:this.$router.push("路由映射的路径")
```
btnfn(){
    this.$router.push("/home")
}
```

## 路由传参
### 第一种  占位符
to=
```
1.在router.js 添加带参数的路径映射
:id 表示 /后为一个id参数
export const routes=[
{path:'/user/:id',component:user}
]

2.路由页面接收参数:
this.$route.params.id

export default{
    data(){
        return {
            id:this.$route.params.id
        }
    }
}

3.使用:
<router-link to="/user/10"></router>

```
### 第二种 带params参数传递
:to=

注意:其中的name 一定是等于路由配置里的name
```
<router-link :to="{name:'/user',params:{id:1}}"></router>

```
### 第三种 js代码方式
```
this.$router.push({name:'/user',params:{id:1}});
```


## 嵌套路由 (子路由)
```
在原先的路由下 配置children标签
这样路径指向就是原来的 /子路径

比如:
export default new Router({
routes:[
    {
    path:'/', //路由路径
    component:home,
    name: 'home'    //路由名称 
    children:[ //子路由配置
      {
      path:'/info', //子路由路径
      component:info,
      name: 'info'    //子路由名称 
      },
    ]
   }
]
})

在home.vue文件下引用的话
就是<router-link to="/info"></router>
注意:不需要带父路径的前缀,不然无法访问
```

## 路由重定向
redirect配置
```
export default new Router({
routes:[
    {
    path:'/', //路由路径
    component:home,
    name: 'home'    //路由名称 
   },
   {
     path:'/test',
     redirect:'/' ,
   }
]
})
添加一个新的路由配置 也就是说 `/test`路径会重定向到`/`这个路径

```

## 路由钩子函数
### 函数概念
```
beforeRouteEnter: 在进入路由前执行
beforeRouteLeave: 在进入路由后执行
(to,from,next):
to: 路由要跳转的路径信息
from: 路由来自的路径信息
next: 是否继续下一步
```
### 函数使用
```
export default:{
    props:['id'],
    name:'UserProfile',
    beforeRouteEnter:(to,from,next){
      console.log("准备进入当前这个vue页面");
      next();
    },
    beforeRouteEnter:(to,from,next){
      console.log("准备离开当前这个vue页面");
      next();
    },
}


```

## 配置404页的路由
这个路由需要放在最底部
```
export default new Router({
routes:[
  {
    path:'/404', //404路由路径
    component:404,  
    name: '404'    //404路由名称 
   },
   {   
    path:'*',    //其他所有路径都转发到404
    redirect:'/404'
   }
]
})
```