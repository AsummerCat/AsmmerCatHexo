---
title: go的sync并发包
date: 2022-10-19 14:07:31
tags: [go]
---
# go的sync并发包

## 计数器sync.WaitGroup
sync.WaitGroup
```
package main

import (
	"fmt"
	"sync"
)

var wg sync.WaitGroup

func main() {
	go test()
	go test()
	test()
	wg.Wait() //等待计数器减为0
	fmt.Println("结束")
}

func test() {
	//新增计数器1
	wg.Add(1)
	for i := 0; i < 10; i++ {
		fmt.Println(i)
	}
	fmt.Println("输出完成")
	//减少计数器
	defer wg.Done()
}

```

## sync.Once 高并发场景下仅执行一次
用于加载文件,或者关闭通道
```
func (o *Once) Do (f func()){}
```
```
package main

import (
	"fmt"
	"sync"
)

/*
SyncOnce 高并发场景下只执行一次 如加载文件或者关闭通道
*/

var loadOnce sync.Once

func main() {
	//入参是方法名称
	loadOnce.Do(test)
}

func test() {
	fmt.Println("测试")
}

```

## sync.Map 并发安全的Map
```
package main

import (
	"fmt"
	"sync"
)

/*
SyncMap  并发安全的Map(开箱即用无需make函数初始化) 内置操作方法
*/

var m2 = sync.Map{}

func main() {

	//存储键值对
	m2.Store("key", "value")

	//查询key的value
	value, ok := m2.Load("key")
	if ok {
		fmt.Println(value)
	}

	//删除key
	m2.Delete("key")

}

```
