---
title: 基于jdk1.8currentHashMap理解
date: 2020-05-11 15:19:07
tags: [java,jdk源码解析,currentHashMap]
---

# 基于jdk1.8currentHashMap理解

1. 从JDK1.7到1.8，ConcurrentHashMap 的结构从ReentrantLock+Segment+HashEntry变化为synchronized+CAS+HashEntry+红黑树。

2.在扩容的情况下，哈希冲突得到一定的处理，提高了效率。

3.在JDK1.8中ConcurrentHashMap和HashMap一样，采取了红黑树结构，和链表选择使用，提高了遍历的速度。

<!--more-->

# **JDK1.7版本的CurrentHashMap的实现原理**

在JDK1.7中ConcurrentHashMap采用了**数组+Segment+分段锁**的方式实现。

ConcurrentHashMap中的**分段锁称为Segment**，它即类似于HashMap的结构，即内部拥有一个Entry数组，数组中的每个元素又是一个链表,同时又是一个ReentrantLock（Segment继承了ReentrantLock）。

ConcurrentHashMap使用分段锁技术，将数据分成一段一段的存储，然后给每一段数据配一把锁，当一个线程占用锁访问其中一个段数据的时候，其他段的数据也能被其他线程访问，能够实现真正的并发访问。如下图是ConcurrentHashMap的内部结构图：

![](/img/2020-05-09/jdk1.7ConcurrentHashMap.jpg)

# JDK1.8版本的CurrentHashMap的实现原理

JDK8中ConcurrentHashMap参考了JDK8 HashMap的实现，采用了**数组+链表+红黑树**的实现方式来设计，内部大量采用CAS操作，JDK8中彻底放弃了Segment转而采用的是Node，其设计思想也不再是JDK1.7中的分段锁思想。

在JDK8中ConcurrentHashMap的结构，由于引入了红黑树，使得ConcurrentHashMap的实现非常复杂，我们都知道，红黑树是一种性能非常好的二叉查找树，其查找性能为O（logN），但是其实现过程也非常复杂，而且可读性也非常差，Doug Lea的思维能力确实不是一般人能比的，早期完全采用链表结构时Map的查找时间复杂度为O（N），JDK8中ConcurrentHashMap在链表的长度大于某个阈值的时候会将链表转换成红黑树进一步提高其查找性能。



