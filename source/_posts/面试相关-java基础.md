---
title: 面试相关-java基础
date: 2020-02-03 20:30:13
tags: [java,面试相关]
---

# 基础知识

### 各个基础类型占用多少字节?

```
类型          占用字节  
byte           1
short          2
int            4      
long           8
float          4
double         8
char           2
boolean        1
```

<!--more-->

### Servlet的生命周期

```
实例化，初始init，接收请求service，销毁destroy
```



## HashMap与ConcurrentHashMap相关

```java
获取hashMap的数组位置   -> i= 根据key的hashCode % table.length-1     = table[i]=entry
```

### HashMap

```
hashMap
数据结构 :   数组+链表     1.8后转换红黑树
链表插入方式: 1.7头插      1.8尾插     
扩容: 在75%的时候扩大1倍         扩容1.7可能会出现死循环
Hash冲突:用链表解决散列冲突
插入: key 和value可为空
多线程可能出现的问题:  
扩容时多线程操作可能会导致链表成环的出现，然后调用 get 方法会死循环
1.7对线程put时,会在超过填充因子的情况下rehash.HashMap为避免尾部遍历,链表插入采用头插法,多线程场景下可能产生死循环.


```

### ConcurrentHashMap

```java
ConcurrentHashMap
数据结构  : 数组+链表   链表大于8并且总容量长度大于64 ->后转换红黑树,链表长度小于6时重新转为一般链表.(8,6,64为默认参数)

         concurrencyLevel =   Segment的个数     用来分段 不指定默认为1段
         initialCapacity  = 	HashEntry的个数
  ===============================================
         结构: 
             Segment[]
                  HashEntry[]
                       链表 or 红黑树
  ===============================================              
扩容: 在75%的时候扩大1倍
并发安全:  分段锁机制 
                1.7中采用segment分段加锁,降低并发锁定程度.
          CAS自旋锁            
                1.8中采用CAS自旋锁(一种乐观锁实现模式)提高性能.但在并发度较高时,性能一般.
  利用Unsafe类的CAS操作直接修改机器内存 
     
插入: key value 不为空  
```

# 集合相关

```
Collection 包  
     List
     Set 
     Stack
     Vector 
Map包
```

### Collection包下有什么

```
     List 
        ArrayList   默认10的数组 扩容 1.5倍 查询快
        LinkedList   增删快 因为链表
     Set 
        hashSet
        TreeSet
        SortedSet
     Vector
```

### hashSet 底层是hashMap

```
因为: hashSet 的key就是添加的那个值 value是默认object常量
```

# 真题

### Collection 和Collections类的区别是什么?

```
Collections类 是集合的工具类
Collection是集合的顶级接口
```

###  Hashtable和HashMap与ConcurrentHashMap的不同是什么?

```
Hashtable 是不允许键或值为 null 的，HashMap 的键值则都可以为 null。
ConcurrentHashMap不允许为null

1.因为在put的时候已经做个判断不为null 否则抛出异常
2.如果为null的话 无法判断是存在还是不存在这个key ,无法再调用一次contain(key）来对key是否存在进行判断
3.实现方式不同：Hashtable 继承了 Dictionary类，而 HashMap 继承的是 AbstractMap 类。
4.初始化容量不同：HashMap 的初始容量为：16，Hashtable 初始容量为：11，两者的负载因子默认都是：0.75。
5.有一个快速失败（fail—fast）的机制 在遍历过程中修改了集合会报错 有维护一个modCount值
```

### 如何解决ABA的问题?

```
ABA 就是原值A 修改为B 又改回A 

这边可以用版本号来控制
```

### == 和 equals 的区别是什么?

```
基本类型使用   ==比较基本类型的值 
引用类型用    ==比较引用地址是否相同 
            equals比较   值
```

### HTTP2和HTTP的区别有哪些?

```
二进制传输    http2采用二进制传输，相较于文本传输的http1来说更加安全可靠。

多路复用     http1一个连接只能提交一个请求，而http2可以同时处理无数个请求，可以降低连接的占用数量，进一步提升网络的吞吐量。

头部压缩     缩小了头部容量，间接提升了传输效率

服务端推送   服务端可以主动推送资源给客户端，避免客户端花过多的时间逐个请求资源，这样可以降低整个请求的响应时间。
```

### http 和 socket 有什么区别和联系

```java
根据 osi 分层，socket 属于传输层，http 属于应用层。

就如 socket 是连接你我的管道，但管道中传送东西流程和规范的最佳实践之一就是 http
```

### **什么情况会造成内存泄漏**

```
在Java中，内存泄漏就是存在一些被分配的对象，这些对象有下面两个特点：

首先，这些对象是可达的，即在有向图中，存在通路可以与其相连；

其次，这些对象是无用的，即程序以后不会再使用这些对象。

如果对象满足这两个条件，这些对象就可以判定为Java中的内存泄漏，这些对象不会被GC所回收，然而它却占用内存。
```

**平时有没有用linux系统，怎么查看某个进程** 

```
ps aux|grep java 查看java进程
ps aux 查看所有进程
ps –ef|grep tomcat 查看所有有关tomcat的进程
ps -ef|grep --color java 高亮要查询的关键字
kill -9 19979 终止线程号位19979的进程
```

### 浏览器输入 URL 发生了什么?

```
1.DNS 解析
2.TCP 连接
3.发送 HTTP 请求
4.服务器处理请求并返回 HTTP 报文
5.浏览器解析渲染页面
6.连接结束
```

### hashmap为什么选择8作为阀值?

```
根据泊松分布 选择阈值为8
根据概率
```

###  JDK动态代理和CGlib代理的差别?

```
jdk动态代理: 需要实现InvocationHandlet接口
CGlib: 字节码增强
```

### 一个service执行多次,生成多少个实例?

```
Servlet的生命周期由Web容器来进行管理
只会生成一个实例

1.实例化，一个Servlet类只会创建一个实例，当第一个用户第一次访问Servlet时，创建对象保存在web容器中；
2.初始化，调用init方法，对Servlet进行初始化，只执行一次
3.服务，调用service方法，用户每访问一次Servlet就调用一次
4.销毁，调用destroy方法，web容器关闭
5.最后，Servlet 是由 JVM 的垃圾回收器进行垃圾回收的。
```

### ArrayList线程不安全?

```
 报错: ConcurrentModificationException
 
 原因: 并发修改错误会造成list中标记的modifyCount 不相等
 
 优化:  1.使用Vector (线程安全)
       2.Collections.synchronizedList 生成
       3.copyOnWriteArrayList 读时复制(读写分离思想 ) add(加锁+复制array[]+1)
       
```

### hashSet底层为什么是hashMap 并且key value是什么?

```
hashSet 的key就是添加的那个值 
value是定义的一个默认object常量
```

###  hashSet线程不安全?

```
跟ArrayList线程不安全一样
```

