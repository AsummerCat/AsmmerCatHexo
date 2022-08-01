---
title: HashMap与HashTable与treeMap比对
date: 2018-10-27 16:36:20
tags: [java]
---

# 对比
* 都是属于Map接口下的
   
* hashMap和TreeMap  继承了AbstractMap抽象类  

* hashMap 初始容量大小16  内容大于0.75时候触发容量大小翻倍 
线程不安全 key可以为null hash会被设置为0 

* TreeMap 是一个key有序的Map  

* hashTable 是线程安全的 但是性能相对较差 key不能为空 会报NPE

*  JDK8中进行了锁的大量优化 在多线程中 可以使用 ConcurrentHashMap 而不是hashMap

<!--more-->

# HashMap与HashTable比对

```
package maptest;

import java.util.*;

/**
 * @author cxc
 * @date 2018/10/27 15:53
 * HashMap与HashTable 区别
 * hashTable 线程安全 synchronized  键值对都不为空 因为key为null  算不出hash 就抛出了NPE
 * hashMap 线程不安全 key可以为null  如果为null hash会被被默认计算为0
 */
public class HashMapAndHashTable {
    public static void main(String[] args) {
        try {
            hashMapAdd();
        } catch (Exception e) {
            e.printStackTrace();
            System.out.println("HashMap操作失败");
        }
        try {
            hashTableAdd();
        } catch (Exception e) {
            e.printStackTrace();
            System.out.println("HashTable操作失败");
        }
    }

    /**
     * 创建hashMap
     */
    private static void hashMapAdd() {
        Map map = new HashMap<>();
        map.put(null, 1);
        Collection values = map.values();
        values.forEach(data -> System.out.println("hashMap的值:" + data.toString()));
        Set set = map.entrySet();
        set.forEach(data -> System.out.println("hashMap的key:" + data));
    }

    /**
     * 创建HashTable
     */
    private static void hashTableAdd() {
        Map map = new Hashtable<>();
        map.put("测试", 1);
        Collection values = map.values();
        values.forEach(data -> System.out.println("HashTable值:" + data.toString()));
        Set set = map.entrySet();
        set.forEach(data -> System.out.println("HashTable的key:" + data));
    }
}
```

# hashMap与treeMap对比

```
package maptest;

import java.util.*;

/**
 * @author cxc
 * @date 2018/10/27 16:46
 * HashMap 的key是无序的
 * treeMap 的key是有序的
 * Set entries = map.entrySet();遍历键值对
 */
public class HashMapAndTreeMap {
public static void main(String[] args){
     hashMapAdd();
     treeMapAdd();
}

    /**
     * hashMap遍历
     */
    private static void hashMapAdd() {
            Random random=new Random();
        HashMap<String,Integer> map=new HashMap<>();
        int num=1000;
        for (int i = 0; i < num; i++) {
            map.put(Integer.toString(random.nextInt(1000)),i);
        }
        Set entries = map.entrySet();
        entries.forEach(data -> System.out.println("hashMap的值:" + data.toString()));

    }

    /**
     * treeMap遍历
     */
    private static void treeMapAdd() {
        Random random=new Random();
        TreeMap<String,Integer> map=new TreeMap<>();
        int num=1000;
        for (int i = 0; i < num; i++) {
            map.put(Integer.toString(random.nextInt(1000)),i);
        }
        Set entries = map.entrySet();
        entries.forEach(data -> System.out.println("treeMap的值:" + data.toString()));
    }
}

```