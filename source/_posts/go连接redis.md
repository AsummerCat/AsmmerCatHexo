---
title: go连接redis
date: 2023-12-07 09:56:37
tags: [go]
---

## go连接redis
`导入依赖: go get github.com/go-redis/redis/v8 `
```
package main

import (
	"context"
	"fmt"
	"github.com/go-redis/redis/v8"
	"time"
)

var (
	rdb *redis.Client
)

// 初始化连接
func initClient() (err error) {
	rdb = redis.NewClient(&redis.Options{
		Addr:     "localhost:16379",
		Password: "",  // no password set
		DB:       0,   // use default DB
		PoolSize: 100, // 连接池大小
	})

	ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
	defer cancel()

	_, err = rdb.Ping(ctx).Result()
	return err
}

func V8Example() {
	ctx := context.Background()
	if err := initClient(); err != nil {
		return
	}

	err := rdb.Set(ctx, "key", "value", 0).Err()
	if err != nil {
		fmt.Println(err)
		return
	}

	val, err := rdb.Get(ctx, "key").Result()
	if err != nil {
		fmt.Println(err)
		return
	}
	fmt.Println("key", val)

	val2, err := rdb.Get(ctx, "key2").Result()
	if err == redis.Nil {
		fmt.Println("key2 does not exist")
	} else if err != nil {
		fmt.Println(err)
		return
	} else {
		fmt.Println("key2", val2)
	}
}

func main() {
	err := initClient()
	if err != nil {
		fmt.Println(err)
		return
	}
	V8Example()
}

```