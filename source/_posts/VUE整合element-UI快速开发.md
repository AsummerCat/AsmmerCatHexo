---
title: VUE整合element-UI快速开发
date: 2020-06-02 19:48:22
tags: [vue,element-UI]
---

# VUE整合elementUI快速开发

## 安装elementUI
```
npm i element-ui -S
```

## 创建项目流程
```
1. 初始化项目
vue init webpack hello-vue
2. 进入项目目录
cd hello-vue
3.安装vue-router
npm install vue-router --save-dev 
4. 安装element-ui
npm i element-ui -S
5. 安装依赖
npm install
6.安装SASS加载器
cnpm install sass-loader node-sass --save-dev
7.启动测试
npm run dev

```

<!--more-->

## 使用element-UI

```
<!--ele导入组件-->
import ElementUI from 'element-ui';
<!--ele导入样式-->
import 'element-ui/lib/theme-chalk/index.css';

使用:
Vue.use(ElementUI);

new Vue({
 el: '#app',
 render: h =>h(App)//ElementUI
 <!-- App 指的是使用的 .vue文件名称-->
})
```