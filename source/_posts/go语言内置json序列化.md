---
title: Go语言内置json序列化
date: 2022-10-19 14:02:16
tags: [go]
---
# Go语言内置json序列化

## 引入依赖
```
import (
	"encoding/json"
)
```

## 序列化和反序列化
```
marshal, err := json.Marshal(person)

	//反序列化
	var p2 Person
	json.Unmarshal([]byte(str), &p2) //传指针修改P2的赋值
```
<!--more-->
## 案例
```
package main

import (
	"encoding/json"
	"fmt"
)

/*
*
注意字段大写 才能在别的包中查看,所以json序列化的时候需要变成大写
*/
type Person struct {
	Name string
	Age  int
}

func main() {
	person := Person{
		Name: "小明",
		Age:  100,
	}
	//序列化
	marshal, err := json.Marshal(person)
	if err != nil {
		fmt.Println("序列化失败:%s", err)
	}

	str := string(marshal)
	fmt.Println(string(marshal))

	//反序列化
	var p2 Person
	json.Unmarshal([]byte(str), &p2) //传指针修改P2的赋值
	fmt.Println(p2)
}

```
