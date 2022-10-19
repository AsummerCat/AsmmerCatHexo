---
title: go语言的gin框架的中间件
date: 2022-10-19 14:21:47
tags: [go,gin]
---
# go语言的gin框架的中间件

可以等价与java中的拦截器,所有请求都需要经过这里

gin中间件必须是一个`gin.HandlerFunc类型`


## 简单中间件案例
```
加载全局中间件

	//1.创建默认路由
	router := gin.Default()
    //加载中间件
	router.Use(handlerAll())
```
```
func handlerAll() gin.HandlerFunc {

	return func(c *gin.Context) {
		fmt.Println("中间件开始执行了")
		//执行的业务 设置变量到context中
		c.Set("request", "中间件")
     	//执行目标函数
		c.Next()
		status := c.Writer.Status()
		fmt.Println("中间件执行完毕", status)
	}
}
```
<!--more-->

## next方法
主要是为了执行目标函数  整个中间件类似java的aop环绕通知

## 局部中间件
可指定特定映射 或者映射组使用该中间件
```
	//8.局部中间件的使用 针对某个映射或者映射组
	router.GET("/handlerTest",AsyncTest, handlerPortion())

```



