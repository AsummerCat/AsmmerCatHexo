---
title: vue笔记登录校验处理
date: 2020-06-25 22:04:18
tags: [vue]
---

# vue笔记登录校验处理
## 在路由前配置beforeEach
来保证拦截住请求

<!--more-->

```
在main.js下配置

router.beforeEach((to,from,next)=>{
    //获取用户登录状态
    let isLogin=sessionStorage.getItem('isLogin');
    
    //注销
    if(to.path=='/logout'){
        //清空sessionStorage
        sessionStorage.clear();
        next({path:'/login'})
    }
    //如果是登录页
    else if(to.path=='/login'){
      //已登录 跳转首页
        if(isLogin!=null){
            next({path:'/home'})
        }
    }else{
        //如果是访问其他页面,必须要求登录后才可以
        if(isLogin!=null){
            next({path:'/home'})
        }
    }
})

```
这样就能做到 每个请求之前拦截 判断是否登录 
不允许登录就能访问的页面就被拦截了