---
title: go语言的pprof资源监控
date: 2022-10-19 14:11:23
tags: [go]
---
# go语言的pprof资源监控

runtime/pprof 采集工具型应用运行数据进行分析  (本地使用)
net/http/pprof 采集服务型应用运行数据进行分析 (线上服务使用)

默认10ms进行采集一次堆栈信息,CPU和内存资源

## 如何解析数据
采集到的监控数据会输出一个`xxx.pprof`文件

可以使用
```
go tool pprof xx.pprof 
```
来获取输出的监控数据

<!--more-->
## 案例
```
package main

import (
	"fmt"
	"os"
	"runtime/pprof"
)

/*
*
内置资源采集工具
比如CPU资源等

runtime/pprof 采集工具型应用运行数据进行分析  (本地使用)
net/http/pprof 采集服务型应用运行数据进行分析 (线上服务使用)

默认10ms进行采集一次堆栈信息,CPU和内存资源
*/
func main() {
	runtimeTest()
}

/*
本地数据分析
*/
func runtimeTest() {
	file, err := os.Create("./CPU.pprof")
	if err != nil {
		fmt.Println("创建采集文件失败")
	}
	//1.开始采集CPU信息
	pprof.StartCPUProfile(file)

	//2.停止采集CPU信息
	defer pprof.StopCPUProfile()

	//3.堆栈信息也写入文件中
	pprof.WriteHeapProfile(file)

	file.Close()

}

```
