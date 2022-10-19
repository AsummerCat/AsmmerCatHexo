---
title: Go语言的context
date: 2022-10-19 14:15:32
tags: [go]
---
# Go语言的context

用来统一规范 和解决 不同协程之间的交互

## 案例
ctx, cancel := context.WithCancel(context.Background())
主要方法:
```
ctx context.Context
//1.获取一个context
ctx, cancel := context.WithCancel(context.Background())
//2.对应方法传入ctx
func worker(ctx context.Context){
    //等待上级通知
    true==ctx.Done()
}
//3. cancel() // 通知子goroutine结束  


```
<!--more-->
```
package main

import (
	"context"
	"fmt"
	"sync"

	"time"
)

var wg sync.WaitGroup

func worker(ctx context.Context) {
LOOP:
	for {
		fmt.Println("worker")
		time.Sleep(time.Second)
		select {
		case <-ctx.Done(): // 等待上级通知
			break LOOP
		default:
		}
	}
	wg.Done()
}

func main() {
	ctx, cancel := context.WithCancel(context.Background())
	wg.Add(1)
	go worker(ctx)
	time.Sleep(time.Second * 3)
	cancel() // 通知子goroutine结束
	wg.Wait()
	fmt.Println("over")
}
```

##  context.WithValue的使用

类似TreadLocal的使用
//赋值
context.WithValue(ctx, TraceCode("TRACE_CODE"), "12512312234")
//获取值
traceCode, ok := ctx.Value(key).(string)

### 案例
```
package main

import (
	"context"
	"fmt"
	"sync"

	"time"
)

/*
 context.WithValue的使用
类似TreadLocal的使用
//赋值
context.WithValue(ctx, TraceCode("TRACE_CODE"), "12512312234")
//获取值
traceCode, ok := ctx.Value(key).(string)
*/

type TraceCode string

var wg1 sync.WaitGroup

func worker1(ctx context.Context) {
	key := TraceCode("TRACE_CODE")
	traceCode, ok := ctx.Value(key).(string) // 在子goroutine中获取trace code
	if !ok {
		fmt.Println("invalid trace code")
	}
LOOP:
	for {
		fmt.Printf("worker, trace code:%s\n", traceCode)
		time.Sleep(time.Millisecond * 10) // 假设正常连接数据库耗时10毫秒
		select {
		case <-ctx.Done(): // 50毫秒后自动调用
			break LOOP
		default:
		}
	}
	fmt.Println("worker done!")
	wg1.Done()
}

func main() {
	// 设置一个50毫秒的超时
	ctx, cancel := context.WithTimeout(context.Background(), time.Millisecond*50)
	// 在系统的入口中设置trace code传递给后续启动的goroutine实现日志数据聚合
	ctx = context.WithValue(ctx, TraceCode("TRACE_CODE"), "12512312234")
	wg1.Add(1)
	go worker1(ctx)
	time.Sleep(time.Second * 5)
	cancel() // 通知子goroutine结束
	wg1.Wait()
	fmt.Println("over")
}

```

## 使用Context的注意事项
```
推荐以参数的方式显示传递Context
以Context作为参数的函数方法，应该把Context作为第一个参数。
给一个函数方法传递Context的时候，不要传递nil，如果不知道传递什么，就使用context.TODO()
Context的Value相关方法应该传递请求域的必要数据，不应该用于传递可选参数
Context是线程安全的，可以放心的在多个goroutine中传递
```
