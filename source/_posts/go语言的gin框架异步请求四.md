---
title: go语言的gin框架异步请求四
date: 2022-10-19 14:21:20
tags: [go,gin]
---
# go语言的gin框架异步请求四

同步请求和异步请求
注意 启动goroutime的时候,不应该使用原始上下文,必须使用它的只读副本
```
context := c.Copy()
```
<!--more-->
## 案例
```
/*
异步请求
*/
func AsyncTest(c *gin.Context) {
	//创建一个context的只读副本
	context := c.Copy()
        go func() {
			//休眠3秒
			time.Sleep(3*time.Second)
			log.Print("异步执行:"+context.Request.URL.Path)
		}()
	c.String(http.StatusOK,"异步请求返回")
}
```
