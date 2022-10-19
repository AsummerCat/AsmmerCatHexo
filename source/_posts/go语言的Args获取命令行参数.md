---
title: go语言的Args获取命令行参数
date: 2022-10-19 14:12:17
tags: [go]
---
# go语言的Args获取命令行参数

os.Args获取命令行参数
注意:
需要 `go build` 后 才可使用
```
PS D:\godemo\day10> .\ArgsTest.exe
[D:\godemo\day10\ArgsTest.exe]

```
<!--more-->
ps: 也可以使用flag来获取参数
## 案例
```
package main

import (
	"flag"
	"fmt"
	"os"
)

func main() {
	fmt.Print("完整参数列表: ")
	fmt.Println(os.Args)
	//获取命令行输出的 参数 按照空格区分
	fmt.Println("第一个参数(当前执行程序地址):" + os.Args[0])
	fmt.Println("第二个参数:" + os.Args[1])
	fmt.Println("第二个参数:" + os.Args[2])

	//也可以使用flag来获取对应坐标位置
	flag.Parse()
	args := flag.Args()
	arg := flag.Arg(0)  //第一个参数
	arg1 := flag.Arg(1) //第二个参数
	fmt.Print("flag获取的所有参数:")
	fmt.Println(args)
	fmt.Println("flag获取的第一个参数:" + arg)
	fmt.Println("flag获取的第二个参数:" + arg1)
	/*
		执行结果
	PS D:\godemo\day10> go build .\ArgsTest.go
	PS D:\godemo\day10> .\ArgsTest.exe param1 param2
	完整参数列表: [D:\godemo\day10\ArgsTest.exe param1 param2]
	第一个参数(当前执行程序地址):D:\godemo\day10\ArgsTest.exe
	第二个参数:param1
	第二个参数:param2
	flag获取的所有参数:[param1 param2]
	flag获取的第一个参数:param1
	flag获取的第二个参数:param2


	*/
}


```
