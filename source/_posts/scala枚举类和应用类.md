---
title: scala枚举类和应用类
date: 2022-08-18 17:45:14
tags: [大数据,scala]
---
# scala枚举类和应用类

枚举类需要继承 :`Enumeration`,
应用类需要继承 :`App`
<!--more-->

## 枚举例子 Enumeration
```
object Color extends Enumeration{
    val RED = Value(1,"red");
    val YELLOW = Value(1,"yellow");
}
```
调用:
```
println(Color.RED) 
输出的是 red 底层存储的是1
```
## 应用例子 App
```
相当于@Test 可以直接运行

object TestApp extends App{
    println("app start") 
}
``

