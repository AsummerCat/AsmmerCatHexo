---
title: go的变量常量定义和主函数运行编译
date: 2022-10-19 13:55:27
tags: [go]
---
# go的变量常量定义和主函数运行

## 编译
```
go build

会在主函数下生成一个对应的exe文件 可直接运行
```
## 运行
```
go run main.go
```
<!--more-->
## 演示代码
```
package main

import "fmt"

// 变量批量声明
var (
	name string
	age  int
	isOk bool
)

// 主函数执行
// 函数外 只能设置标识符(变量,常量,函数,类型)的声明
func main() {
	fmt.Println("hello Word")
	name = "小明"
	age = 18
	isOk = true
}

```


## 变量

### 数字类型
| 字段类型 | 备注 |
| --- | --- |
| uint8 | 无符号 8 位整型 (0 到 255) |
| uint16 | 无符号 16 位整型 (0 到 65535) |
| uint32 | 无符号 32 位整型 (0 到 4294967295) |
|uint64 |无符号 64 位整型 (0 到 18446744073709551615) |
| int8| 有符号 8 位整型 (-128 到 127)|
|int16 |有符号 16 位整型 (-32768 到 32767) |
|	int32 |有符号 32 位整型 (-2147483648 到 2147483647) |
| int64|有符号 64 位整型 (-9223372036854775808 到 9223372036854775807) |
|byte|类似 uint8|
|rune|类似 int32|
|uint|32 或 64 位|
|int|与 uint 一样大小|



### 浮点型
| 字段类型 | 备注 |
| --- | --- |
|float32 | IEEE-754 32位浮点型数|
| float64| IEEE-754 64位浮点型数|
|complex64 | 32 位实数和虚数|
|complex128 |64 位实数和虚数 |

## 变量声明
```
局部变量声明 在func内部

例如:

var age=18
```
```
全局变量声明
func方法外部

例如:
// 变量批量声明
var (
	name string
	age  int
	isOk bool
)
```

## 常量声明
```
    const LENGTH int = 10
	const WIDTH int = 5
```

