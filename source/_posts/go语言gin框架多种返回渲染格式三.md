---
title: go语言gin框架多种返回渲染格式三
date: 2022-10-19 14:20:53
tags: [go,gin]
---
# go语言gin框架多种返回渲染格式

## 返回json数据
```
/*
返回json数据
*/
func resJson(c *gin.Context) {
	badJson := gin.H{"name": "小明", "age": 18}
	//以json的形式返回
	c.JSON(http.StatusOK, badJson)

	//结构体响应
	var msg struct {
		Name   string
		Number int
	}
	c.JSON(http.StatusOK, msg)
}
```
<!--more-->

## 返回html模板数据
1.首先需要在src同级目录里建立`templates`模板目录
2.新增index.html
```
<html>
<body>
<h1>
    {{.title}}
</h1>
</body>
</html>
```
变量: ` {{.title}}`表示可以由返回值进行替换
3. 在路由加入模板目录
```
	//1.创建默认路由
	router := gin.Default()
	//限制表单上传大小
	router.MaxMultipartMemory = 8 << 20
	//加载模板文件
	router.LoadHTMLGlob("templates/*")
```
4. 调用c.html返回
#### 案例
```
/*
返回html模板数据
*/
func resHtml(c *gin.Context) {
	//1.首先需要在路由那边加载模板文件
	/*
		router := gin.Default()
		//加载模板文件
		router.LoadHTMLGlob("templates/*")
	*/
	//最终json将title替换
	c.HTML(http.StatusOK, "index.html", gin.H{"title": "我的标题"})
}
```

## 重定向
函数:`c.Redirect`
```
/*
重定向
*/
func redirectTest(c *gin.Context) {
	//状态码 301 重定向地址 支持内部地址和外部地址
	c.Redirect(http.StatusMovedPermanently, "http://www.baidu.com")
	//c.Redirect(http.StatusMovedPermanently, "/")
}
```
