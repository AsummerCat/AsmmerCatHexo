---
title: linux系统监控命令
date: 2019-03-30 15:48:55
tags: linux
---

# 系统监控类的命令

## uptime  查看系统基本情况

![top命令](/img/2019-3-30/uptime.png)

```java
uptime
--系统时间

--运行时间
  * 案例中为 开机后运行 6 mins
  
--连接数
  * 每一个终端算一个连接 1 users
  
--1,5,15分钟内系统平均负载
   * 运行队列中的平均负载  数值越大 负载越高
```



<!--more-->

## top 查看进程基本情况

![top命令](/img/2019-3-30/top.png)

```java
top 
 --最上面几行同 uptime
 
 --下面是 cpu /内存情况
 
 --列表
  每个进程占cpu的情况

```



## vmstat 统计系统的cpu,内存,swap,io等

![top命令](/img/2019-3-30/vmstat.png)

可以带参数  例如: `vmstat 1 4 ` 采样频率/s,采样总条数

```
vmstat
 注意:这个命令无法在mac上运行
 
--cpu占用率很高,上下文切换频繁,说明系统又线程正在频繁切换
cpu中的 cs us 上下文切换频率


内容说明
io 中的 bi :输入   bo:输出
cpu中的 cs us: 上下文切换频率   us :用户占用的cpu

```



## 111

