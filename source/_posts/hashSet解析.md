---
title: hashSet解析
date: 2019-02-13 10:25:05
tags: jdk源码解析
---

# hashSet

`该容器中只能存储不重复的对象`

```java
public class HashSet<E>
    extends AbstractSet<E>
    implements Set<E>, Cloneable, java.io.Serializable
```

public boolean add(Object o)方法用来在Set中添加元素，当元素值重复时则会立即返回false，如果成功添加的话会返回true。

底层还是使用hashMap实现的

这边就是将值 作为hashMap的key  利用一个定义的虚拟的object对象当做全部key的value

<!--more-->

## 构造方法摘要

编辑

`HashSet()`

构造一个新的空 set，其底层 HashMap 实例的默认初始容量是 16，加载因子是 0.75。

`HashSet(Collection<? extends E> c)`

构造一个包含指定 collection 中的元素的新 set。

`HashSet(int initialCapacity)`

构造一个新的空 set，其底层 HashMap 实例具有指定的初始容量和默认的加载因子（0.75）。

`HashSet(int initialCapacity, float loadFactor)`

构造一个新的空 set，其底层 HashMap 实例具有指定的初始容量和指定的加载因子。



## 参数

// 使用 HashMap 的 key 保存 HashSet 中所有元素

`private transient HashMap<E,Object> map;`

// 定义一个虚拟的 Object 对象作为 HashMap 的 value

`private static final Object PRESENT = new Object();`



## 方法摘要

编辑

`boolean add(E e)`

如果此 set 中尚未包含指定元素，则添加指定元素。

`void clear()`

从此 set 中移除所有元素。

`Object clone()`

返回此 HashSet 实例的栈表副本：并没有复制这些元素本身。

`boolean contains(Object o)`

如果此 set 包含指定元素，则返回 true。

`boolean isEmpty()`

如果此 set 不包含任何元素，则返回 true。

`Iterator<E> iterator()`

返回对此 set 中元素进行迭代的迭代器。

`boolean remove(Object o)`

如果指定元素存在于此 set 中，则将其移除。

`int size()`

返回此 set 中的元素的数量（set 的容量）。

从类 java.util.AbstractSet 继承的方法

`equals, hashCode, removeAll`

从类 java.util.AbstractCollection 继承的方法

`addAll, containsAll, retainAll, toArray, toArray, toString`

从类 java.lang.Object 继承的方法

`finalize, getClass, notify, notifyAll, wait, wait, wait`

从接口 java.util.Set 继承的方法

`addAll, containsAll, equals, hashCode, removeAll, retainAll, toArray, toArray`

