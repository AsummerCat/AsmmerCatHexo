---
title: vue-router路由
date: 2020-06-02 19:49:12
tags: [vue]
---

# vue-router路由

## 安装vue-router

```
npm install vue-router --save-dev
```
### 使用:
 ```
import Vue from 'vue'
import VueRouter from 'vue-fouter'

<!--显示声明使用 -->
Vue.use(VueRouter);
 ```
<!--more-->

## 配置路由地址

在/src/router/ 新建index.js

```
import Vue from 'vue'
import VueRouter from 'vue-router'

//安装路由
Vue.use(VueRouter);

//配置导出路由

export default new VueRouter({
    routes: [
    
    {
     //路由地址
     path:'/content',
     //路由名称
     name: 'Content',
     //跳转到组件
     component: Content
    },{
      //路由地址
     path:'/main',
     //路由名称
     name: 'Main',
     //跳转到组件
     component: Main
    }
    
    ]
    
});

```

### 在main.js中配置路由信息
main.js写法
```
import Vue from 'vue'
import App from 'app'
//自动扫描里面的路由配置
import router from './router'

new Vue({
    el: '#app',
    //配置全局的路由
    router,
    component: { App },
    template: '<App/>'
})

```

### 在其他页面如何调用
```
跳转链接
<router-link to="/main">跳转到某个链接</router-link>

跳转视图
<router-view to="/main">跳转到页面</router-view>
```

### 嵌套路由
```
在index.js中
export default new VueRouter({
    routes: [
    
    {
     //路由地址
     path:'/content',
     //路由名称
     name: 'Content',
     //跳转到组件
     component: Content,
    <!--嵌套路由--> //跳转到组件
     children: [
     {path:'/user/pag1', component: pag1},
      {path:'/user/pag2', component: pag2}
     ]
    }
]
    
});

```
### 传递参数  params
```
方式一:
<router-link to="{name:'/user/profile',params:{id:1}}"></router-link>

//传递属性后 ->下一个页面取用:
{{$route.params.属性}}

方式二:
路由上配置: props:true

接收页面使用:
export default{
    props: ['id']
}

{{id}}
```

## 重定向:

在router的index.js配置中写入:
```
{
   <!--路径配置-->
    path: '/goHome',
    <!--重定向到那个位置redirect属性-->
    redirect: '/main'
}
```

## 设置路由模式移除地址栏的#号
```
在router的index.js配置中写入:
export default new VueRouter({
    routes: [
    //修改路由模式 hash: 路径带#号 ,history不带#号
    mode:'history',
    {
     //路由地址
     path:'/content',
     //路由名称
     name: 'Content',
     //跳转到组件
     component: Content
    }
    ]
    
});

```

## 定义自定义的404页面

1.创建404的 .vue文件
2.配置路由
```
{
payh: '*',
component: NotFound
}
```

## 自定义钩子函数

beforeRouteEnter :在进入路由前执行
beforeRouteLeave: 在离开路由前执行

```
使用:
export default {
    props: ['id'],
    name: "UserProfile",
    beforeRouteEnter: (to,from,next)=>{
        console.log("准备进入个人信息页");
        next();
    },
    beforeRouteLeave: (to,from,next)=>{
        console.log("准备离开个人信息页");
        next();
    }
    
    
}
```
```
函数里面的参数详解:
to:路由简要跳转的路径信息
from:路径跳转前的路径信息
next: 路由的控制参数
  * next()跳入下一个页面
  * next('/path')改变路由的跳转方向,时期跳到另外一个路由
  * next(false)返回原来的页面
  * next((vm)=>{})仅在beforeRouteEnter中可用,vm是组件实例
```

## 钩子配合Axios 可实现预加载数据

```


export default {
    props: ['id'],
    name: "UserProfile",
    beforeRouteEnter: (to,from,next)=>{
        console.log("准备进入个人信息页");
        //路由之前->加载数据
        next(vm=>{
        //可调用组件
           vm.getData();
        });
    },
    methods: {
        getData: function(){
          this.axios({
             method: 'get',
             url: 'http://baidu.com/data.json'
          }).then((response){
          console.log("获取到数据");
          );  
        }
    }
}
    

```