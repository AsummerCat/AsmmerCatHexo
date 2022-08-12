---
title: Flume进阶和架构四
date: 2022-08-12 09:42:02
tags: [Flume,大数据]
---
# Flume的架构
### Flume Agent内部原理
![image](/img/2022-08-01/Flume.png)

<!--more-->

# Flume进阶事务的使用

开启事务后会在Source后 有一个临时缓冲区putList

![image](/img/2022-08-05/1.png)