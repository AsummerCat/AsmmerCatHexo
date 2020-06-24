---
title: vue笔记使用webpack模式构建并安装依赖
date: 2020-06-24 22:28:38
tags: [vue,webpack]
---

# vue笔记使用webpack模式构建并安装依赖
并且安装依赖

## 构建项目
```
vue init webpack xxx

注意的是:测试模块和严格模式都不需要打开
```
![](/img/2020-06-19/7.png)

## 安装插件
```
# 安装vue-router路由模块
npm install vue-router --save-dev

# 安装element-ui模块
npm i element-ui -S

# 安装SAAS加速器 (前端css升级版本)
npm install sass-loader node-sass --save-dev

```
<!--more-->