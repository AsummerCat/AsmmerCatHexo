---
title: go语言的gin框架的cookie和session
date: 2022-10-19 14:22:10
tags: [go,gin]
---
# go语言的gin框架的cookie和session

## cookie和session
cookie 客户端浏览器缓存

session 服务器缓存

<!--more-->

## 相关API
```
//设置cookie
c.SetCookie()
//获取cookie
c.Cookie()


//获取session

//设置session
```

## cookie案例
```
func setCookie(c *gin.Context) {
	//设置cookie
	//maxAge int 单位秒
	//path,cookie所在目录
	//domian string 域名
	//secure 是否只能通过https访问
	//httpOnly bool 是否允许别人通过js获取自己的cookie
	c.SetCookie("name_cache", "小明", 60, "/", "localhost", false, true)
}

func getCookie(c *gin.Context) {
	//获取客户端是否携带cookie
	cookie, err := c.Cookie("name_cache")
	if err != nil {
		log.Print("暂未获取到cookie:", err)
	}
	fmt.Println(cookie)
}
```
