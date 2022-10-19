---
title: go语言依赖下载位置
date: 2022-10-19 14:15:03
tags: [go]
---
# go语言依赖下载位置

## 配置 GO111MODULE

GO111MODULE环境变量主要是⽤来开启或关闭模块⽀持的。
它有三个可选值：off、on、auto，默认值是 auto。
GO111MODULE=off ⽆模块⽀持，go 会从 GOPATH 和 vendor ⽂件夹寻找包。
GO111MODULE=on 模块⽀持，go 会忽略 GOPATH 和 vendor ⽂件夹，只根据 go.mod 下载依赖。
GO111MODULE=auto 在 $GOPATH/src 外⾯且根⽬录有 go.mod ⽂件时，开启模块⽀持。
在使⽤模块的时候，GOPATH 是⽆意义的，不过它还是会把下载的依赖储存在 $GOPATH/src/mod 中，也会把 go install 的结果放在
$GOPATH/bin 中。
<!--more-->
## windows 设置go module
go env -w GO111MODULE=auto

## go mode命令
常用的`go mod`命令如下

| 命令 | 含义 |
| --- | --- |
| go mod download | 下载依赖的module到本地cache(默认为$GOPATH/pkg/mod目录) |
| go mod edit|  编辑go.mod文件|
| go mod graph| 打印模块依赖图 |
| go mod init| 初始化当前文件夹,创建go.mod文件  |
| go mod tidy | 增加缺少的module,删除无用的module |
| go mod vendor| 将依赖复制到vendor下  |
| go mod verify| 检验依赖 |
| go mod why| 解释为什么需要依赖 |


