---
title: java的监控工具
date: 2019-03-30 16:57:33
tags: [java,jvm]
---

# jps 列出java进程



```
命令: jps

- 列出java进程类似ps命令

参数:
  -q :可以指定jps只输出进程ID,不输出类的短名称
  -m :可以用于输出传递给java进程主函数main的参数
  -i :可以用于输出主函数的完整路径
  -v :可以显示传递给jvm的参数
```



# jinfo 用来查看java应用程序扩展参数 修改部分参数

![jinfo](/img/2019-3-30/jinfo.png)

```
命令: jinfo

- 用来查看java应用程序扩展参数 修改部分参数 (简单的 比如 -PringGCDetail)

命令格式:  jinfo -flag xxxx pid 

参数:
 -flag  <name> 打印指定jvm参数
 -flag +-<name> 设置指定jvm参数的布尔值
 -flag <name>=<value> 设置指定jvm参数的值
 
```



<!--more-->

# jmap 用来生成快照和对象统计信息

```
命令: jmap
--用来生成快照和对象统计信息
操作:1.打印出对象占用信息到文件  jmap -histo pid > d:\a.txt
    2.打印dump到文件          jmap -dump:format=b,file=c:\heap.hprof pid
    3.显示当前gc占用          jmap -heap pid

```



# jstack  查看锁情况 打印线程dump

```java
命令 jstack 
--
操作 : 1.强制dump，当jstack没有响应时使用   jstack -F pid 
      2.查看 打印锁信息                   jstack -l pid
      3.查看 打印java和native的帧信息      jstack -m pid
      4. 打印dump                        jstack 120 >>C:\a.txt
```



# JConsole 图形化监控工具

```
可以查看Java应用程序的运行概况，监控堆信息、永久区使用情况、类加载情况等

```



# Visual VM 功能强大的多合一故障诊断和性能监控的可视化工具

```

Visual VM是一个功能强大的多合一故障诊断和性能监控的可视化工具


```

