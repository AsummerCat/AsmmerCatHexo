---
title: vue.js路由
date: 2018-11-09 09:13:12
tags: vue.js
---
# Vue.js路由(Vue-router)

* [官方文档](https://cn.vuejs.org/v2/guide/routing.html#%E5%AE%98%E6%96%B9%E8%B7%AF%E7%94%B1)
*  大意就是 用来替代跳转 因为现在变成模块化了 每个都是组件

# 使用
## 导入js

<!--more-->

```
<script src="vue.js"></script>
<script src="vue-router.js"></script>
```

如果在一个模块化工程中使用它，必须要通过 Vue.use() 明确地安装路由功能：
在你的文件夹下的 src 文件夹下的 main.js 文件内写入以下代码

```
import Vue from 'vue'
import VueRouter from 'vue-router'

Vue.use(VueRouter)
```

## 定义模板

可以理解为html页面

```
<!--定义模版-->
<template id="a">
    <div>
        第一个router
    </div>
</template>
<template id="b">
    <div>
        第二个router
    </div>
</template>
```

## js

```
var routes = [
    {
    //表示html路径
        path:"/one",
    //表示名称
		 name:"one",
	// 具体哪个组件
        component:one
    },
    {
        path:"/two",
        name:"two",
        component:two
    },
];

// 定义路由组件
var router = new VueRouter({
    routes
});


// 定义路由
new Vue({
    el:"#box",
    router
});
// 创建和挂载实例
```

## 将模版增添链接

```
<div id="box"> 
    <router-link to="/one">One</router-link>
    <router-link to="/two">Two</router-link>
      <!-- 路由匹配到的组件将渲染在这里 -->
  <router-view></router-view>
</div>

```

# Router-link跳转路径的书写的几种方式

```
直接书写路径<router-link to="/about"></router-link>


动态指定<router-link :to="{path:’about’}"></router-link>


:to ====v-bind:to
通过name动态绑定，防止路径过长<router-link :to="{name:’about’,params:{userId:123}}"></router-link>
```

通过注入路由器，我们可以在任何组件内通过 `this.$router` 访问路由器，也可以通过 `this.$route` 访问当前路由


# 动态路由匹配

```
var routes = [
    {
    //表示html路径   动态匹配
        path:"/one/:id",
    //表示名称
		 name:"one",
	// 具体哪个组件
        component:one
    }
  ]
    
这样的话 无论是 /one/小明 还是 /one/小东 都会跳到这个路由
 

// 定义路由组件
var router = new VueRouter({
    routes
});
```

一个“路径参数”使用冒号 : 标记。当匹配到一个路由时，参数值会被设置到 `this.$route.params`，可以在每个组件内使用。于是，我们可以更新 User 的模板，输出当前用户的 ID：

```
{{ $route.params.id }}

```
你可以在一个路由中设置多段“路径参数”，对应的值都会设置到 `$route.params `中。例如：

<table><thead><tr><th>模式</th> <th>匹配路径</th> <th>$route.params</th></tr></thead> <tbody><tr><td>/user/:username</td> <td>/user/evan</td> <td><code>{ username: 'evan' }</code></td></tr> <tr><td>/user/:username/post/:post_id</td> <td>/user/evan/post/123</td> <td><code>{ username: 'evan', post_id: 123 }</code></td></tr></tbody></table>

---

# 路由组件传参

## 页面接收参数

* 这里需要注意的是:<font color="red">接受参数：用$route，少个r,注意啦</font>

## 第一种 params
// 一定要写name,params必须用name来识别路径

```
<router-link :to="{ name: 'user', params: { userId: 123 }}">User</router-link>

这跟代码调用 router.push() 是一回事：

router.push({ name: 'user', params: { userId: 123 }})

```
<font color="red">如果路由组件配置的路径是 `path:"user/:id"`  两种方式都会把路由导航到 /user/123 路径

如果路由组件配置的路径是 `path:"user"`  两种方式都会把路由导航到 `/user` 路径</font>
## params页面接收参数

```

{{this.$route.params.userId}}
```

## 第二种 query 
查询参数其实就是在路由地址后面带上参数和传统的url参数一致的，传递参数使用query而且必须配合path来传递参数而不能用name，目标页面接收传递的参数使用query。

```
this.$router.push({ path: '/news', query: { userId: 123 }});
```

## query页面接收参数

```
{{this.$route.query.userId}}
```

最后总结：路由传递参数和传统传递参数是一样的，命名路由类似表单提交而查询就是url传递，在vue项目中基本上掌握了这两种传递参数就能应付大部分应用了，最后总结为以下三点：  

* <font color="red">1.命名路由搭配params，刷新页面参数会丢失</font>
* <font color="red">2.查询参数搭配query，刷新页面数据不会丢失</font>
* <font color="red">3.接受参数使用this.$router后面就是搭配路由的名称就能获取到参数的值</font>

---

# 嵌套路由

```
 children: [{
 	//因为是嵌套的 所以直接给路径就可以了
    path: 'emails',
    component: UserEmailsSubscriptions
  }
  
```
实例:

```
var routes = [
    {
    //表示html路径
        path:"/one",
    //表示名称
		 name:"one",
	// 具体哪个组件
        component:one,
        children: [{
 	//因为是嵌套的 所以直接给路径就可以了
    path: 'emails',
    component: UserEmailsSubscriptions
  }
    }
  ]
    
这样的话 /one/emails都会跳到这个嵌套的路由
 

// 定义路由组件
var router = new VueRouter({
    routes
});
```


---
