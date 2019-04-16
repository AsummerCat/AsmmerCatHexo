---
title: 使用guava实现根据IP的令牌桶
date: 2019-04-16 15:01:16
tags: [java,guava]
---

# 使用guava实现根据IP的令牌桶

## 导入pom.xml

```
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>27.1-jre</version>
</dependency>
```

## RateLimiter

guava中提供了令牌桶的一个封装实现RateLimiter, 可以直接调用, 省的我们自己包装ConcurrentHashMap + Timer.

我们预设的场景是服务器端提供一个API供不同客户端查询, 要限流每个IP每秒只能调用两次该API.

<!--more-->

### 首先要定义一个服务器端的缓存, 定期清理即可, 缓存 IP : 令牌桶

```
// 根据IP分不同的令牌桶, 每天自动清理缓存
    private static LoadingCache<String, RateLimiter> caches = CacheBuilder.newBuilder()
            .maximumSize(1000)  //最大缓存上限
            .expireAfterWrite(1, TimeUnit.DAYS) // 1天未被写剔除
            .build(new CacheLoader<String, RateLimiter>() {
                @Override
                public RateLimiter load(String key) throws Exception {
                    // 新的IP初始化 (限流每秒两个令牌响应)
                    return RateLimiter.create(2);
                }
            });


```

### 然后在业务代码中进行限流调用

```
private static void login(int i) throws ExecutionException {
        // 模拟IP的key
        String ip = String.valueOf(i).charAt(0) + "";
        RateLimiter limiter = caches.get(ip);

        if (limiter.tryAcquire()) {
            System.out.println(i + " success " + new SimpleDateFormat("HH:mm:ss.sss").format(new Date()));
        } else {
            System.out.println(i + " failed " + new SimpleDateFormat("HH:mm:ss.sss").format(new Date()));
        }
    }


```

### 模拟客户端调用

```
   for (int i = 1000; ;) {//模拟同一个ip请求多次
            // 模拟实际业务请求
            Thread.sleep(100);
            login(i);
        }


```

# 完整代码

```
package org.jeecgframework.test.demo;

import com.google.common.cache.*;
import com.google.common.util.concurrent.RateLimiter;

import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.TimeUnit;

public class guavatest {

    private static LoadingCache<String, RateLimiter> caches = CacheBuilder.newBuilder().
            maximumSize(1000)    //最大缓存上限
            .concurrencyLevel(8)  //设置并发级别为8，并发级别是指可以同时写缓存的线程数 默认 4
            .expireAfterAccess(10, TimeUnit.SECONDS)  ///设置时间对象没有被读/写访问则对象从内存中删除
            // .expireAfterWrite(1,TimeUnit.DAYS) ////设置时间对象没有被写访问则对象从内存中删除
            .recordStats()//设置要统计缓存的命中率
            .removalListener(new RemovalListener<Object, Object>() {

                @Override
                public void onRemoval(RemovalNotification<Object, Object> notification) {
                    System.out.println(notification.getKey() + "被移除");
                }
            })
            //CacheLoader类 实现自动加载
            //new CacheLoader<String, "自定义返回值 标识 get(key) 获取到的类型">
            .build(new CacheLoader<String, RateLimiter>() {
                @Override
                public RateLimiter load(String key) throws Exception {
                    // 新的IP初始化 (限流每秒两个令牌响应)
                    System.out.println(key + "初始化");

                    return RateLimiter.create(2);
                }
            });


    public static void main(String[] args) throws Exception {

        for (int i = 1000; ; ) {//模拟同一个ip请求多次
            // 模拟实际业务请求
            Thread.sleep(100);
            login(i);
        }

    }

    private static void login(int i) throws ExecutionException {
        // 模拟IP的key
        String ip = String.valueOf(i).charAt(0) + "";
        RateLimiter limiter = caches.get(ip);

        if (limiter.tryAcquire()) {
            System.out.println(i + " success " + new SimpleDateFormat("HH:mm:ss.sss").format(new Date()));
        } else {
            System.out.println(i + " failed " + new SimpleDateFormat("HH:mm:ss.sss").format(new Date()));
        }
        //最后打印缓存的命中率等 情况
        System.out.println(caches.stats().toString());
    }
}

```