---
title: Go的原子操作Atomic包
date: 2022-10-19 14:08:11
tags: [go]
---
# Go的原子操作Atomic包
原子操作
<!--more-->

## Atomic包

```
package main

import (
	"fmt"
	"sync/atomic"
)

/*
*
原子包 atomic包
原子操作
*/
func main() {
	var x int64 = 200

	//存储原子值 注意这里存储的是指针
	atomic.AddInt64(&x, 1)
	//加载原子值
	loadInt64 := atomic.LoadInt64(&x)
	fmt.Println(loadInt64)
	//atomic.xxx其他类型
}

```
