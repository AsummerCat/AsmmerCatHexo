---
title: go的实体结构体
date: 2022-10-19 13:58:36
tags: [go]
---
# go的实体结构体

## 操作
```
/* 定义结构体 */
type Circle struct {
	radius float64
}
```
## 代码
```
package main

import "fmt"

/* 定义结构体 */
type Circle struct {
	radius float64
}

type Books struct {
	title string
	author string
	subject string
	book_id int
}


// 该 method 属于 Circle 类型对象中的方法
func (c Circle) getArea() float64 {
	//c.radius 即为 Circle 类型对象中的属性
	return 3.14 * c.radius * c.radius
}

func main() {
	var c1 Circle
	c1.radius = 10.00
	fmt.Println("圆的面积 = ", c1.getArea())

	// 创建一个新的结构体
	fmt.Println(Books{"Go 语言", "www.runoob.com", "Go 语言教程", 6495407})
}


```