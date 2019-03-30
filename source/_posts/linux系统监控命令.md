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



## pidstat 细致观察进程(需要安装)

![pidstat命令](/img/2019-3-30/pidstat.png)

可以带参数  例如: ` pidstat p 进程号 u 1 3`   -p +进程号  -u表示:查看cpu的值  1:采集频率 3:采集条数

![例子](/img/2019-3-30/pidstat1.png)

```
pidstat 
需要安装插件 一般来说 云服务器上都有 如果没有手动安装
安装命令: sudo apt-get install sysstat
 作用: 
  -- 监控cpu -u
  -- 监控io -d
  -- 监控内存 
  -- 监控线程 -t
  
  一般来说使用这个 `参数 -t` 用来监控进程下的线程 搭配上面例子使用 
```

