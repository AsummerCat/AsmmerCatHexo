---
title: Go语言的socket通信
date: 2022-10-19 14:08:53
tags: [go]
---
# Go语言的socket通信
## 引入相关包
```
import (
	"net"
)
```
<!--more-->
## 服务端
```
package main

import (
	"bufio"
	"fmt"
	"net"
)

/**
socket服务端
*/

func main() {
	//开启端口服务
	listen, err := net.Listen("tcp", "127.0.0.1:20000")
	if err != nil {
		fmt.Println("监听端口失败", err)
	}

	for {
		//等待客户端建立连接
		conn, err := listen.Accept()
		if err != nil {
			fmt.Println("连接失败", err)
			continue
		}
		//处理连接的通信
		go process(conn)
	}
}

/*
处理函数
*/
func process(conn net.Conn) {
	//关闭连接
	defer conn.Close()
	for {
		reader := bufio.NewReader(conn)
		var buf [128]byte
		n, err := reader.Read(buf[:]) //读取数据
		if err != nil {
			fmt.Println("读取客户端数据失败", err)
			break
		}
		recvStr := string(buf[:n])
		fmt.Println("收到client端发来的数据:", recvStr)
		conn.Write([]byte(recvStr)) //发送数据
	}
}

```
## 客户端
```
package main

import (
	"fmt"
	"net"
	"time"
)

/*
*
socket客户端
*/
func main() {
	//1.与server建立连接
	client, err := net.Dial("tcp", "127.0.0.1:20000")
	if err != nil {
		fmt.Println("连接127.0.0.1:20000失败", err)
	}

	for {
		//2.发送数据
		client.Write([]byte("hello world"))

		time.Sleep(time.Duration(2) * time.Second)
	}
	//3.关闭连接
	client.Close()
}

```
