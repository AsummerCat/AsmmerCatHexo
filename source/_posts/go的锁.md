---
title: go的锁
date: 2022-10-19 14:05:25
tags: [go]
---
# go的锁
sync包下的容
<!--more-->
## 互斥锁
var lock sync.Mutex
```
package main

import (
	"fmt"
	"sync"
)

// 创建一个全局锁
var lock sync.Mutex

/*
互斥锁
*/
func main() {
	tryLock := lock.TryLock()
	if tryLock {
		fmt.Println("已锁定")
	} else {
		fmt.Println("未锁定")
	}
	lock.Lock()

	lock.Unlock()
}


```

## 读写锁
rwlock sync.RWMutex
```
package main

import (
	"sync"
)

// 创建一个读写锁
var (
	rwlock sync.RWMutex
)

/*
读写锁
*/
func main() {
	//读锁
	rwlock.RLock()
	//解除读锁
	rwlock.RUnlock()
	//写锁
	rwlock.Lock()
	//解除写锁
	rwlock.Unlock()
}

```


