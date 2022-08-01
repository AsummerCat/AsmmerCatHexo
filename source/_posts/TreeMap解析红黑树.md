---
title: TreeMap解析红黑树
date: 2019-02-13 11:06:15
tags: [jdk源码解析]
---

参考:[[Java提高篇（二七）-----TreeMap](https://www.cnblogs.com/chenssy/p/3746600.html)](https://www.cnblogs.com/chenssy/p/3746600.html)

参考[Java 集合系列12之 TreeMap详细介绍(源码解析)和使用示例](https://www.cnblogs.com/skywang12345/p/3310928.html)

# TreeMap(未完成 后期学习红黑树 )

## TreeMap的定义如下

```
public class TreeMap<K,V>
    extends AbstractMap<K,V>
    implements NavigableMap<K,V>, Cloneable, java.io.Serializable
```

## **TreeMap 简介**

* TreeMap 是一个**有序的key-value集合**，它是通过[红黑树](http://www.cnblogs.com/skywang12345/p/3245399.html)实现的。
* TreeMap **继承于AbstractMap**，所以它是一个Map，即一个key-value集合。
* TreeMap 实现了NavigableMap接口，意味着它**支持一系列的导航方法。**比如返回有序的key集合。
* TreeMap 实现了Cloneable接口，意味着**它能被克隆**。
* TreeMap 实现了java.io.Serializable接口，意味着**它支持序列化**。

TreeMap基于**红黑树（Red-Black tree）实现**。该映射根据**其键的自然顺序进行排序**，或者根据**创建映射时提供的 Comparator 进行排序**，具体取决于使用的构造方法。
TreeMap的基本操作 containsKey、get、put 和 remove 的时间复杂度是 log(n) 。
另外，TreeMap是**非同步**的。 它的iterator 方法返回的**迭代器是fail-fastl**的。



## 构造函数

```
// 默认构造函数。使用该构造函数，TreeMap中的元素按照自然排序进行排列。
TreeMap()

// 创建的TreeMap包含Map
TreeMap(Map<? extends K, ? extends V> copyFrom)

// 指定Tree的比较器
TreeMap(Comparator<? super K> comparator)

// 创建的TreeSet包含copyFrom
TreeMap(SortedMap<K, ? extends V> copyFrom)
```

