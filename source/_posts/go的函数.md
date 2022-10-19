---
title: go的函数
date: 2022-10-19 13:58:05
tags: [go]
---
# go的函数

## function
```
package main

import "fmt"

func main() {
	/* 定义局部变量 */
	var a int = 100
	var b int = 200
	var ret int

	/* 调用函数并返回最大值 */
	ret = max(a, b)

	fmt.Printf( "最大值是 : %d\n", ret )
}

/* 函数返回两个数的最大值 */
func max(num1, num2 int) int {
	/* 定义局部变量 */
	var result int

	if (num1 > num2) {
		result = num1
	} else {
		result = num2
	}
	return result
}
```
<!--more-->

## 返回多个值
```
func swap(x, y string) (string, string) {
   return y, x
}

func main() {
   a, b := swap("Google", "Runoob")
   fmt.Println(a, b)
}
```

## 对象的方法demo
```
package main

import "fmt"



type person struct {
	Name string
	age  int
}

/**
对象的方法 前面的参数表示调用方是哪个
*/

func (p person) getName() {
	fmt.Println(p.Name)
}
func main() {
	p1 := person{Name: "小明", age: 11}
	p1.getName()
}

```
