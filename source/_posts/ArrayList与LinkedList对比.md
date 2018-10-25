---
title: ArrayList与LinkedList对比
date: 2018-10-24 14:25:37
tags: [java]
---

#测试环境
MacBook Pro (13-inch, 2017, Two Thunderbolt 3 ports)  
2.3 GHz Intel Core i5
8 GB 2133 MHz LPDDR3

---

# jdk内存 
-server -Xms2048m -Xmx2048m -Xss512k

---

<!--more-->


# 介绍
基本上理解是:

```
ArrayList和LinkedList的大致区别如下:
1.ArrayList是实现了基于动态数组的数据结构，LinkedList基于链表的数据结构。 
2.对于随机访问get和set，ArrayList觉得优于LinkedList，因为LinkedList要移动指针。 
3.对于新增和删除操作add和remove，LinedList比较占优势，因为ArrayList要移动数据。 

ArrayList:
      数据结构:基于动态数组
      扩容:默认初始值为10 ,扩容为原先的1.5倍
      长度:可变
      顺序增加:快
      随机访问:快 (因为有序,直接查找索引就可以了)
      增加删除:慢 (因为要移动数组)
LinkedList:
       数据结构:基于双向链表 (前驱 后继)
       扩容: 没有扩容的概念(只要有前驱和后继就可以++++)
       顺序增加:慢 
       随机访问:慢(因为需要查找该元素前驱和后续 没有就反向查找就比较慢了)
       增加删除:快 
```

# 测试

具体测试数据:
可能测速数据有误差 

## 顺序在结尾插入

```
 结论: ArrayList插入速度比LinkedList快

500w数据量

1. ArrayList没有初始值
ArrayList插入5000000记录耗时141毫秒
LinkList插入5000000记录耗时174毫秒

2.ArrayList有初始值
ArrayList插入5000000记录耗时87毫秒
LinkList插入5000000记录耗时238毫秒


第一种 因为ArrayList是动态添加的数组 不设定初始值的话  
就在创建的时候,超出了他的初始值 每次扩容变成1.5倍 消耗时间

```

## 在集合头部插入

```
结论: LinkedList插入速度比ArrayList 明显快多了

10w数据
ArrayList插入100000记录耗时616毫秒
LinkList插入100000记录耗时10毫秒


50w数据
ArrayList插入500000记录耗时14564毫秒
LinkList插入500000记录耗时29毫秒


因为ArrayList插入的时候需要重写排序,  
而linkedList只需要加上前驱和后继就可以了,效率明显快多了

```

## 随机插入 

```
结论:   ArrayList插入速度比LinkedList明显快多了

5w数据
ArrayList插入50000记录耗时84毫秒
LinkedList插入50000记录耗时4190毫秒


```

## 查找:

```
ArrayList 比 LinkedList 快 

5w数据:
ArrayList查找全部50000记录耗时0毫秒
LinkedList查找全部50000记录耗时1毫秒

因为 ArrayList 有索引
    而linkedList 只能通过顺次指针访问

```

## 删除:

```
LinkedList 比 ArrayList 快

5w数据:
ArrayList删除全部50000记录耗时115毫秒
LinkedList删除全部50000记录耗时4毫秒


 因为:  linkedList改变前驱和后继就可以了  
       ArrayList需要移动数组
```