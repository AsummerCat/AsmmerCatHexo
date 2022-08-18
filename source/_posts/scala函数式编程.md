---
title: scala函数式编程
date: 2022-08-18 17:43:35
tags: [大数据,scala]
---

# scala函数式编程

## 基本语法
```
 def sum ( x : Int , y : Int ) :Int ={
     x+y
 }
```
<!--more-->

## 方法传参添加默认值
//如无传参,则使用默认值
```
 def f2(num :String ="默认值" ): Unit ={
     print(num)
 }
```
 