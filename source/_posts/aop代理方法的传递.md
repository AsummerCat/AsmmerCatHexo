---
title: aop代理方法的传递
date: 2020-04-08 21:30:40
tags: [spring,java,aop]
---

# aop代理方法的传递

```
默认在 aop方法中 同类下的方法调用不继续调用

this.Amethod()是默认调用本类
AopContext.currentProxy()是调用代理类实现
```
比如:

```
method A(){
    this.B方法 
    this.C方法
}

这样的话 切面代理的是A方法

```
<!--more-->

下面是:各自方法使用代理类实现

```
method A(){
   ((Service) AopContext.currentProxy()).B方法 
   ((Service) AopContext.currentProxy()).C方法
}

使用((Service) AopContext.currentProxy()) 这个来保证该方法是使用代理类实现的
这样 aop和事务都能保证在各个业务上

```


## 需要开启这个配置

```
<aop:aspectj-autoproxy proxy-target-class="true"expose-proxy="true"/>  
```