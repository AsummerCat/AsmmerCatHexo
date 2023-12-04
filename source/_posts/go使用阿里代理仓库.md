---
title: go使用阿里代理仓库
date: 2023-12-04 15:52:09
tags: [go]
---
# 阿里云Go Module代理仓库服务

go module公共代理仓库，代理并缓存go模块。你可以利用该代理来避免DNS污染导致的模块拉取缓慢或失败的问题，加速你的构建。

<!--more-->
```
1.使用go1.11以上版本并开启go module机制

2.导出GOPROXY环境变量

export GOPROXY=https://mirrors.aliyun.com/goproxy/
```