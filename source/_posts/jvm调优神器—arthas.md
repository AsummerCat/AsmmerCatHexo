---
title: jvm调优神器—arthas
date: 2021-02-07 16:55:44
tags: [java,虚拟机调优,arthas]
---

直接下载一个可以启动的jar包然后用java -jar的方式启动
```
java -jar arthas-boot.jar 
```
启动后 选择应用,下载相关包后就可以执行了

## 相关命令


### thread 获取线程信息
可查看CPU过高,或者死锁信息
```
通过thread加线程id输出当前线程的栈信息
```
查询死锁
```
thread -b 
```

## dashboard 查看监控面板
可以查看内存使用情况
<!--more-->

如果内容使用率在不断上升，而且gc后也不下降，后面还发现gc越来越频繁，很可能就是内存泄漏了。

这个时候我们可以直接用heapdump命令把内存快照dump出来，作用和jmap工具一样
```
[arthas@23581]$ heapdump --live /root/jvm.hprof
Dumping heap to /root/jvm.hprof...
Heap dump file created
```
然后把得到的dump文件，用MAT插件分析就行了
[MAT下载地址](https://www.eclipse.org/mat/downloads.php)

## quit 退出arthas

```
quit
```

## sc 查看jvm中加载的类的信息
查看jvm中加载的类的信息
例如:
```
sc -d java.util.Stack
```

## monitor 监控方法的使用情况
监控方法的使用情况（调用次数、执行时间、失败率等）
```
monitor com.chl.arthes.TestArthas first 
```

##  火焰图
```
[arthas@10031]$ profiler start
Started [cpu] profiling

.... 等待一会后执行
[arthas@10031]$ profiler stop
profiler output file: /Users/chenhailong/eclipse-workspace2/MutilThreadTest/arthas-output/20200408-144819.svg
```
OK
生成的文件通过浏览器 http://127.0.0.1:8563/arthas-output/20200408-144819.svg 可访问。

火焰图，百度自己搜索吧。现在我还不清楚怎么看。

## jvm 查看当前jvm信息
查看当前jvm信息