---
title: vue笔记静态资源打包
date: 2020-06-24 22:30:26
tags: [vue,webpack]
---

# 静态资源打包

## vue的style标签中的scoped属性 
```
如果在vue组件里没有带scoped属性  
那么这个style的样式将会作用在页面中
而不是当前子组件中


比如:

<style scoped>
   div{
       border:1px solid red;
   }
</style>
```
 <!--more-->

## 资源打包
进入开发者模式: `npm run dev`
打包: `npm run build`
```

会在项目根目录生成一个/dist的文件夹
里面是一个打包后的js

结构是:
/dist/
    /生成的js文件 
    /生成的js.map  (这个类似爬虫的map,可以用来vue.js 加速定位资源)

```