---
title: go语言的单元测试
date: 2022-10-19 14:10:42
tags: [go]
---
# go语言的单元测试

## 注意
执行命令`go test`
所有go文件以`_test.go`结尾的源文件在打包的时候都不会打进去
默认为测试类

## 函数命名
```
测试函数 Test开头

基准(性能函数) Benchmark开头

示例函数 Example开头
```

<!--more-->

## 测试函数的格式
每个测试函数比如导入`testing`包,基本结构如下
```
package main

import (
	"fmt"
	"testing"
)

/**
测试函数
*/

func TestName(t *testing.T) {
	//...
	fmt.Println("1")
	t.Errorf("用例失败")
}

```
