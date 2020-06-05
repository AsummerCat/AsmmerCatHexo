---
title: ThreadLocal底层解析
date: 2020-06-06 04:49:40
tags: [java,ThreadLocal]
---

# ThreadLocal底层解析

## 常用方法
###  获取数据
```
  public T get() {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null) {
            ThreadLocalMap.Entry e = map.getEntry(this);
            if (e != null) {
                @SuppressWarnings("unchecked")
                T result = (T)e.value;
                return result;
            }
        }
        return setInitialValue();
    }
```
### 赋值

<!--more-->

```
 public void set(T value) {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null)
          //this是ThreadLocal对象   
            map.set(this, value);
        else
            createMap(t, value);
    }
```

## 底层解析
```
他的内部也是维护一个map 用来存值和获取值

该map是指的 当前线程 存放的value也是当前线程的内容

总体结构:

map: 表示当前线程的Map ->Thread的threadLocals  
key: ThreadLocal对象 value: 当前线程存入的值

```

## 在set的过程中
存入map的是一个Entry 他是继承弱引用的 
垃圾回收 一定会被清理的
```
 static class Entry extends WeakReference<ThreadLocal<?>> {
            /** The value associated with this ThreadLocal. */
            Object value;

            Entry(ThreadLocal<?> k, Object v) {
                super(k);
                value = v;
            }
        }
```
也就是说一个弱引用指向的是 key

为什么要用弱引用 ->因为可能造成内存泄露

## 为什么调用后要remove()
```
因为回收会 软引用  ->垃圾回收后
key等于null,value还存在 无法被访问到
所以依然存在内存泄露

所以!! 一定不用之后一定要调用remove();

```