---
title: SpringBoot整合Ehcache实现缓存功能
date: 2018-10-08 23:34:31
tags: [SpringBoot,Ehcache]
---

>参考:  
>1.[Spring Boot整合Ehcache实现缓存功能](https://blog.csdn.net/Lammonpeter/article/details/78602862)  
>2.[SpringBoot集成ehcache](https://blog.csdn.net/zhangxing52077/article/details/73511694)


# 导入pom.xml

```
 <!-- 缓存依赖 -->
      <dependency>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-starter-cache</artifactId>
      </dependency>

        <dependency>
            <groupId>net.sf.ehcache</groupId>
            <artifactId>ehcache</artifactId>
        </dependency>
```

---

<!--more-->

# 在启动类上加入  开启缓存注解

`@EnableCaching`

---

# 在resources目录下创建ehcache.xml文件

```
<?xml version="1.0" encoding="UTF-8"?>
<ehcache xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="http://ehcache.org/ehcache.xsd">

    <diskStore path="java.io.tmpdir/ehcache"/>

    <!-- 设定缓存的默认数据过期策略 -->
    <defaultCache
            maxElementsInMemory="1000"
            eternal="false"
            overflowToDisk="true"
            timeToIdleSeconds="10"
            timeToLiveSeconds="20"
            diskPersistent="false"
            diskExpiryThreadIntervalSeconds="120"/>

    <cache name="users"
           maxElementsInMemory="1000"
           eternal="false"
           overflowToDisk="true"
           timeToIdleSeconds="1"
           timeToLiveSeconds="20"/>

</ehcache>

```

> <!-- eternal：true表示对象永不过期，此时会忽略timeToIdleSeconds和timeToLiveSeconds属性，默认为false -->
    <!-- maxEntriesLocalHeap：堆内存中最大缓存对象数，0没有限制 -->
    <!-- timeToIdleSeconds： 设定允许对象处于空闲状态的最长时间，以秒为单位。当对象自从最近一次被访问后，
    如果处于空闲状态的时间超过了timeToIdleSeconds属性值，这个对象就会过期，EHCache将把它从缓存中清空。
    只有当eternal属性为false，该属性才有效。如果该属性值为0，则表示对象可以无限期地处于空闲状态 -->

---


# 在application.properities中配置

`#ehcache配置`  
`spring.cache.jcache.config=classpath:ehcache.xml`

---

# 随便写一个vo类

略

---

# 使用缓存注解

```
@CacheConfig(cacheNames = "emp")
    @Cacheable(key = "'user_'+#id")
         @CacheEvict(key = "'user_'+#id")
             @CachePut(key = "'user_'+#id")


```


