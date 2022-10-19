---
title: go语言gin框架数据绑定二
date: 2022-10-19 14:18:48
tags: [go]
---
# go语言gin框架数据绑定

## 定义路由组
```
	//4.数据解析和绑定相关
	bindGroup := router.Group("bind")
	{
		bindGroup.POST("bindJson", bindJson)
		bindGroup.POST("bindFormData", bindFormData)
		bindGroup.POST("/:id/:name/:age", bindFormUrl)
	}
```

## 首先定义接收的结构体
```
// 设置接收多种渠道不一样的定义
type User struct {
	//binding:"required" 定义是 前端必须传递
	Id   int    `form:"id" json:"id" uri:"id" binding:"required"`
	Name string `form:"name" json:"name" uri:"name"`
	Age  int
}
```
<!--more-->
## json数据解析和绑定
```
/*
json数据的解析和绑定
*/
func bindJson(c *gin.Context) {
	//声明接收的变量
	var jsonData User
	//设置绑定到json   将request的body中的数据,自动按照json格式解析到结构体
	err := c.ShouldBindJSON(&jsonData)
	if err != nil {
		//使用gin.H生成json数据,返回json数据
		badJson := gin.H{"error": err.Error()}
		//返回错误信息,以json的形式返回
		c.JSON(http.StatusBadRequest, badJson)
		return
	}

	fmt.Println(jsonData.Id)
	fmt.Println(jsonData.Name)

}
```


## 表单数据解析和绑定
```
/*
表单数据绑定
*/
func bindFormData(c *gin.Context) {
	//声明接收的变量
	var formData User
	//Bind()默认解析bind绑定的form格式
	err := c.Bind(&formData)
	if err != nil {
		//使用gin.H生成json数据,返回json数据
		badJson := gin.H{"error": err.Error()}
		//返回错误信息,以json的形式返回
		c.JSON(http.StatusBadRequest, badJson)
		return
	}
	fmt.Println(formData.Age)
	fmt.Println(formData.Name)
}
```

## URI数据解析和绑定
这里指的是 比如 http://127.0.0.1/user/param1/param2 这种 URL
```
/*
url数据绑定
*/
func bindFormUrl(c *gin.Context) {
	//声明接收的变量
	var formUrl User
	//Bind()默认解析bind绑定的form格式
	err := c.ShouldBindUri(&formUrl)
	if err != nil {
		//使用gin.H生成json数据,返回json数据
		badJson := gin.H{"error": err.Error()}
		//返回错误信息,以json的形式返回
		c.JSON(http.StatusBadRequest, badJson)
		return
	}
	fmt.Println(formUrl.Age)
	fmt.Println(formUrl.Name)
}
```


