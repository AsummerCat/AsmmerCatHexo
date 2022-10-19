---
title: go语言gin框架基础使用一
date: 2022-10-19 14:18:04
tags: [go,gin]
---
# go语言gin框架基础使用
gin是go的一个微服务框架(国内开发的)
* 对于golang而言,web框架的依赖要远比py,java之类的要小.
* 自身的net/http足够简单,性能也非常不错

demo地址:https://github.com/AsummerCat/ginDemo.git

## 安装
```
go get -v github.com/gin-gonic/gin
```
<!--more-->
## 默认路由 简单demo
gin.Conetxt ,封装了request和response
```
package main

import (
	"github.com/gin-gonic/gin"
	"net/http"
)

func main() {
	//1.创建默认路由
	router := gin.Default()
	//2.绑定路由规则,执行函数
	router.GET("/", func(c *gin.Context) {
		//可通过contxt.Query获取带参数的路由
		param := c.Query("userName")
		//返回浏览器 状态码,输出
		c.String(http.StatusOK, "获取参数:"+param+"\n")
		c.String(http.StatusOK, "Hello！欢迎来到GO世界！\n")
		c.String(http.StatusOK, "请求路径:"+c.Request.URL.Path+"\n")
		c.String(http.StatusOK, "请求ip:"+c.ClientIP())
	})

	router.GET("/:name", func(c *gin.Context) {
		//可通过contxt.Query获取带参数的路由
		param := c.Param("name")
		//返回浏览器 状态码,输出
		c.String(http.StatusOK, "获取参数:"+param+"\n")
	})

	//绑定路由规则 不加函数,表示空白页
	router.POST("/xxxPost")
	router.GET("/xxxPost")
	router.PUT("/xxx")
	// 3.默认端口是8080,也可以指定端口 r.Run(":80")
	router.Run()
}


 
```

### API参数
可以通过Context.Param方法来获取
比如 localhost:8080/xxx/xiaoming
```
//注意: :name是映射匹配    使用*name是模糊匹配

	router.GET("/:name", func(c *gin.Context) {
		//可通过contxt.Query获取带参数的路由
		param := c.Param("name")
		//返回浏览器 状态码,输出
		c.String(http.StatusOK, "获取参数:"+param+"\n")
	})
```
返回:
```
请求路径: http://localhost:8080/xiaoming
获取参数:xiaoming
```
### URL参数 也就是 http://localhost:8080/xxx?userName=1
http://localhost:8080/userName=1
```
router.GET("/", func(c *gin.Context) {
		//可通过contxt.Query获取带参数的路由
         param := c.Query("userName")
        //可通过contxt.DefaultQuery如果无参数则给默认值
		param1 := c.DefaultQuery("userName","默认值")
		//返回浏览器 状态码,输出
		c.String(http.StatusOK, "获取参数:"+param+"\n")
		c.String(http.StatusOK, "获取参数:"+param1+"\n")
	})
```

### 表单参数
表单传输为post请求
可以通过`PostForm()`方法获取,该方法默认解析的是
`x-www-form-urlencoded`或`from-data`格式参数
```
//路由
router.POST("/testPost",testPost)

/**
获取表单参数
 */
func testPost(c *gin.Context){
	//获取表单参数
	form := c.PostForm("name")
	c.String(http.StatusOK,"获取表单参数:"+form)
}
```

### 上传文件
注意:服务器限制表单上传文件大小
```
//1.创建默认路由
	router := gin.Default()
	//限制表单上传大小
	router.MaxMultipartMemory = 8 << 20
	//多文件上传
	router.POST("uploadMultipart", uploadMultipart)
	//单文件上传
	router.POST("upload", upload)
```

```
/*
单文件上传
*/
func upload(c *gin.Context) {
	//从表单中提取文件
	file, _ := c.FormFile("file")
	log.Print(file.Filename)
}

/*
多文件上传
*/
func uploadMultipart(c *gin.Context) {
	fileList := c.Request.MultipartForm

	//读取其中的文件
	files := fileList.File["files"]
	for _, file := range files {
		log.Print(file.Filename)
	}
}
```

# 路由分组 group
规范 用来区分get请求和post请求 或者对应模块内容
```
//3.分组函数  类似mvc的前缀 访问路径变更为   http://127.0.0.1/load开头
	uploadGroup := router.Group("load")
	{
		//多文件上传
		uploadGroup.POST("uploadMultipart", uploadMultipart)
		//单文件上传
		uploadGroup.POST("upload", upload)
	}
	
```
