---
title: go并发编程
date: 2022-10-19 14:01:50
tags: [go]
---

# go并发编程

goroutine 是轻量级线程，goroutine 的调度是由 Golang 运行时进行管理的。
## go 函数名( 参数列表 )
```
go f(x, y, z)
```

<!--more-->
Go 允许使用 go 语句开启一个新的运行期线程， 即 goroutine，以一个不同的、新创建的 goroutine 来执行一个函数。 同一个程序中的所有 goroutine 共享同一个地址空间。

## 案例
```
package main

import (
	"fmt"
	"time"
)

func say(s string) {
	for i := 0; i < 5; i++ {
		time.Sleep(100 * time.Millisecond)
		fmt.Println(s)
	}
}
func main() {
	go say("world")
	go say("xxx")
	say("hello")
}

```

## 通道
相当于把多个gotime的结果通过通道输出

通道（channel）是用来传递数据的一个数据结构。

通道可用于两个 goroutine 之间通过传递一个指定类型的值来同步运行和通讯。操作符 <- 用于指定通道的方向，发送或接收。如果未指定方向，则为双向通道。

```
ch <- v    // 把 v 发送到通道 ch
v := <-ch  // 从 ch 接收数据
           // 并把值赋给 v
```

声明一个通道很简单，我们使用chan关键字即可，通道在使用前必须先创建：
```
ch := make(chan int)
```
### 案例
```
package main

import "fmt"
/**
ch <- v    // 把 v 发送到通道 ch
v := <-ch  // 从 ch 接收数据
 */
func sum(s []int, c chan int) {
	sum := 0
	for _, v := range s {
		sum += v
	}
	c <- sum // 把 sum 发送到通道 c
}

func main() {
	s := []int{7, 2, 8, -9, 4, 0}

	c := make(chan int)
	d := make(chan int)
	go sum(s[:len(s)/2], c)
	go sum(s[len(s)/2:], d)
	x, y := <-c, <-d // 从通道 c 中接收

	fmt.Println(x, y, x+y)
}
```

## 通道缓冲区
通道可以设置缓冲区，通过 make 的第二个参数指定缓冲区大小：
```
ch := make(chan int, 100)
```
### 案例
```
package main

import "fmt"

func main() {
    // 这里我们定义了一个可以存储整数类型的带缓冲通道
        // 缓冲区大小为2
        ch := make(chan int, 2)

        // 因为 ch 是带缓冲的通道，我们可以同时发送两个数据
        // 而不用立刻需要去同步读取数据
        ch <- 1
        ch <- 2

        // 获取这两个数据
        fmt.Println(<-ch)
        fmt.Println(<-ch)
}
```


