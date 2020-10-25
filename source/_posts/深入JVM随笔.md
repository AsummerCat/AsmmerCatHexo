---
title: 深入JVM随笔
date: 2020-02-27 16:00:32
tags: [jvm,java]
---

## Java对象布局

```
一个对象:
   1.对象头 (固定)   64位=12byte 32位=4byte
   2.实例变量 
   3.填充数据  ->64位JVM下 对象的内存占用必须是8的倍数 对象头和实例变量不足成为8的倍数情况下填充
```

<!--more-->

## 对象头是什么?

```
定义: 
  ->每个GC管理的堆对象开头的公共结构
  
组成:
   1.Mark Word(标志位)  64bit
     ->存储对象的hashCode,锁信息或者分代年龄或GC标志等信息
     
   2.Class Metadata Address/klass pointer(类元数据地址)   32bit 
     ->类型指针指向对象的类元数据,JVM通过这个指针确认该对象是哪个类的实例
```

## 使用synchronized关键字下的对象有5种状态

```
1.无状态
  -> 刚刚 new出来
2.偏向锁状态
  ->加锁
3.轻量锁状态
4.重量锁状态
5.gc标记
```

## 多线程->立即睡眠和唤醒

```

LockSupport.park()   立即睡眠

LockSupport.unpark(线程)  唤醒线程
```

