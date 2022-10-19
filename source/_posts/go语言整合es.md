---
title: go语言整合es
date: 2022-10-19 14:16:20
tags: [go]
---
# go语言整合es

## 下载依赖
注意下载与你的ES相同版本的client，例如我们这里使用的ES是7.2.1的版本，那么我们下载的client也要与之对应为github.com/olivere/elastic/v7。

使用go.mod来管理依赖：
```
require (
    github.com/olivere/elastic/v7 v7.0.4
)
```
<!--more-->

## 简单示例
```
package main

import (
	"context"
	"fmt"

	"github.com/olivere/elastic/v7"
)

// Elasticsearch demo

type Person struct {
	Name    string `json:"name"`
	Age     int    `json:"age"`
	Married bool   `json:"married"`
}

func main() {
	client, err := elastic.NewClient(elastic.SetURL("http://192.168.1.7:9200"))
	if err != nil {
		// Handle error
		panic(err)
	}

	fmt.Println("connect to es success")
	p1 := Person{Name: "rion", Age: 22, Married: false}
	put1, err := client.Index().
		Index("user").
		BodyJson(p1).
		Do(context.Background())
	if err != nil {
		// Handle error
		panic(err)
	}
	fmt.Printf("Indexed user %s to index %s, type %s\n", put1.Id, put1.Index, put1.Type)
}
```

更多使用详见文档：https://godoc.org/github.com/olivere/elastic
