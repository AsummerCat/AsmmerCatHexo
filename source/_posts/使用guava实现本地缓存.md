---
title: 使用guava实现本地缓存
date: 2019-04-16 15:35:36
tags: [缓存,guava]
---

# 使用guava实现本地缓存

## 创建一个用户类

```
package org.jeecgframework.test.demo;

public class user {
    private String name;
    private Integer age;

    public user(String name, Integer age) {
        this.name = name;
        this.age = age;
    }

    @Override
    public String toString() {
        return "user{" +
                "姓名='" + name + '\'' +
                ", 年龄=" + age +
                '}';
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }
}


```

<!--more-->

## 初始化本地缓存

```
  private static LoadingCache<String, user> caches = CacheBuilder.newBuilder().
            maximumSize(1000)    //最大缓存上限
            .concurrencyLevel(8)  //设置并发级别为8，并发级别是指可以同时写缓存的线程数 默认 4
            .expireAfterAccess(10, TimeUnit.SECONDS)  ///设置时间对象没有被读/写访问则对象从内存中删除
            // .expireAfterWrite(1,TimeUnit.DAYS) ////设置时间对象没有被写访问则对象从内存中删除
            .recordStats()//设置要统计缓存的命中率

            //剔除缓存的监听事件
            .removalListener(new RemovalListener<Object, Object>() {

                @Override
                public void onRemoval(RemovalNotification<Object, Object> notification) {
                    System.out.println(notification.getKey() + "被移除");
                }
            })
            //CacheLoader类 实现自动加载
            //new CacheLoader<String, "自定义返回值  获取标识 get(key) 获取到的类型">
            .build(new CacheLoader<String, user>() {
                @Override
                public user load(String key) throws Exception {
                    System.out.println(key + "写入缓存");
                    return new user("小明" + key, (int) (1 + Math.random() * 10));
                }
            });

```

## 调用缓存方法

```
  private static void login(int i) throws ExecutionException {
        // 模拟IP的key
        String ip = String.valueOf(i).charAt(0) + "";

        //调用本地缓存(没有数据 就回去load)
        user users = caches.get(ip);
        //输出用户
        System.out.println(users.toString());

        //最后打印缓存的命中率等 情况
        //  System.out.println(caches.stats().toString());
    }

```

## 测试类

```
  public static void main(String[] args) throws Exception {
        //模拟用户登录
        for (int i = 1000; i > 0; i--) {
            // 模拟实际业务请求
            Thread.sleep(100);
            login(i);
            i--;
        }
    }

```

------

# 完整代码

```
package org.jeecgframework.test.demo;

import com.google.common.cache.*;

import java.util.concurrent.ExecutionException;
import java.util.concurrent.TimeUnit;

public class guavatest {


    private static LoadingCache<String, user> caches = CacheBuilder.newBuilder().
            maximumSize(1000)    //最大缓存上限
            .concurrencyLevel(8)  //设置并发级别为8，并发级别是指可以同时写缓存的线程数 默认 4
            .expireAfterAccess(10, TimeUnit.SECONDS)  ///设置时间对象没有被读/写访问则对象从内存中删除
            // .expireAfterWrite(1,TimeUnit.DAYS) ////设置时间对象没有被写访问则对象从内存中删除
            .recordStats()//设置要统计缓存的命中率

            //剔除缓存的监听事件
            .removalListener(new RemovalListener<Object, Object>() {

                @Override
                public void onRemoval(RemovalNotification<Object, Object> notification) {
                    System.out.println(notification.getKey() + "被移除");
                }
            })
            //CacheLoader类 实现自动加载
            //new CacheLoader<String, "自定义返回值  获取标识 get(key) 获取到的类型">
            .build(new CacheLoader<String, user>() {
                @Override
                public user load(String key) throws Exception {
                    System.out.println(key + "写入缓存");
                    return new user("小明" + key, (int) (1 + Math.random() * 10));
                }
            });


    public static void main(String[] args) throws Exception {
        //模拟用户登录
        for (int i = 1000; i > 0; i--) {
            // 模拟实际业务请求
            Thread.sleep(100);
            login(i);
            i--;
        }
    }

    private static void login(int i) throws ExecutionException {
        // 模拟IP的key
        String ip = String.valueOf(i).charAt(0) + "";

        //调用本地缓存(没有数据 就回去load)
        user users = caches.get(ip);
        //输出用户
        System.out.println(users.toString());

        //最后打印缓存的命中率等 情况
        //  System.out.println(caches.stats().toString());
    }
}


```