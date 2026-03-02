---
title: Go的基础项目结构
date: 2022-10-19 14:17:17
tags: [go,gin]
---
# Go的基础项目结构
##  结构
```
创建完成后，我们还需要在项目根目录下手动创建 bin，pkg和src三个文件夹。

bin目录：用来存放编译后的exe二进制文件。

pkg目录：用来存放自定义包，也就是import的来源。

src目录：用来存放项目源文件，也就是我们的开发文件。
```
<!--more-->
## 这里需要配置2个系统变量，一个是GOROOT，一个是GOPATH
```
GOROOT指的是go的安装目录,需要指定到bin目录下

GOPATH 是指go的工作目录 (依赖下载位置)
```

注意：GOROOT和GOPATH不能在同一路径下，且变量名必须是GOROOT和GOPATH.


## go.mod
类似pom
用来存储依赖

## 设置go代理
配置go公共代理镜像，目的是解决github无法访问或者访问速度慢的问题
```
go env -w GOPROXY=https://goproxy.io,direct
```
