---
title: Go语言的http服务端和客户端
date: 2022-10-19 14:09:27
tags: [go]
---
# Go语言的http服务端和客户端

## 服务端
可以简单理解为 springmvc
<!--more-->
```
package main

import (
	"fmt"
	"net/http"
)

/**
内置http请求工具 编写HTTP服务端 类似springMvc
*/

func main() {
	//创建一个接口
	http.HandleFunc("/test", f1)
	//开启服务器
	err := http.ListenAndServe("127.0.0.1:8080", nil)
	if err != nil {
		fmt.Println("服务器启动异常", err)
	}

}

func f1(w http.ResponseWriter, r *http.Request) {
	str := "<h1>hello go</h1>"
	w.Write([]byte(str))
}

```

## 客户端
连接指定服务访问接口
```
package main

import (
	"fmt"
	"net/http"
)

/**
内置http请求工具 编写HTTP客户端 连接访问
*/

func main() {

	//get请求

	get, err := http.Get("http://www.baidu.com")
	defer get.Body.Close()
	if err != nil {
		fmt.Println("访问失败", err)
	}
	fmt.Println("访问成功,状态码", get.StatusCode)

	//post请求
	post, err := http.Post("http://www.baidu.com", "application/json", nil)
	defer post.Body.Close()
	if err != nil {
		fmt.Println("访问失败", err)
	}
	fmt.Println("访问成功,状态码", post.StatusCode)

}


```


