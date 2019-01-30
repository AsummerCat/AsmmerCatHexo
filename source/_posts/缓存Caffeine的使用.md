---
title: 缓存Caffeine的使用
date: 2019-01-23 14:18:19
tags: 缓存
---

# 咖啡因

Spring5中推荐使用的缓存  

# 注解

注解在Spring中的应用很广泛，几乎成为了其标志，这里说下使用注解来集成缓存。 
cache方面的注解主要有以下5个

@Cacheable 触发缓存入口（这里一般放在创建和获取的方法上）
@CacheEvict 触发缓存的eviction（用于删除的方法上）
@CachePut 更新缓存且不影响方法执行（用于修改的方法上，该注解下的方法始终会被执行）
@Caching 将多个缓存组合在一个方法上（该注解可以允许一个方法同时设置多个注解）@CacheConfig 在类级别设置一些缓存相关的共同配置（与其它缓存配合使用）

<!--more-->

```java
 /**
     * Cacheable
     * value：缓存key的前缀。
     * key：缓存key的后缀。
     * sync：设置如果缓存过期是不是只放一个请求去请求数据库，其他请求阻塞，默认是false。
     */
```



```
@CachePut(value = "people", key = "#person.id")
```

# 与SpringBoot整合 导入pom

```java
<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-cache</artifactId>
		</dependency>
		<!--咖啡因-->
		<dependency>
			<groupId>com.github.ben-manes.caffeine</groupId>
			<artifactId>caffeine</artifactId>
			<version>2.6.2</version>
		</dependency>

```

# 配置文件

两种方式  

有两种方法：
- application.yml配置文件中配置：
  - 优点：简单
  - 缺点：无法针对每个cache配置不同的参数，比如过期时长、最大容量
- 配置类中配置
  - 优点：可以针对每个cache配置不同的参数，比如过期时长、最大容量
  - 缺点：要写一点代码

### 方式一: 直接写在application.yml中

```java
server:
  port: 8080
spring:
  application:
    name: caffeine-Test
  cache:
    type: caffeine
    cache-names:
      - getA
      - getB
    caffeine:
      spec: maximumSize=500,expireAfterWrite=5s
     
     最后在启动类加入开启缓存的注解
     以上配置创建一getA和getB缓存，最大数量为500，存活时间为5s：
     
@EnableCaching
      
```

### 方式二:在java中使用config

```java
package com.linjing.caffeine;

import com.github.benmanes.caffeine.cache.Caffeine;
import org.springframework.cache.CacheManager;
import org.springframework.cache.annotation.EnableCaching;
import org.springframework.cache.caffeine.CaffeineCache;
import org.springframework.cache.support.SimpleCacheManager;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;

import java.util.ArrayList;
import java.util.concurrent.TimeUnit;

@Configuration
@EnableCaching
public class CacheConfig {

    public static final int DEFAULT_MAXSIZE = 50000;
    public static final int DEFAULT_TTL = 10;

    /**
     * 定義cache名稱、超時時長（秒）、最大容量
     * 每个cache缺省：10秒超时、最多缓存50000条数据，需要修改可以在构造方法的参数中指定。
     */
    public enum Caches {
        getA(5), //有效期5秒
        getSomething, //缺省10秒
        getB(300, 1000), //5分钟，最大容量1000
        ;

        Caches() {
        }

        Caches(int ttl) {
            this.ttl = ttl;
        }

        Caches(int ttl, int maxSize) {
            this.ttl = ttl;
            this.maxSize = maxSize;
        }

        private int maxSize = DEFAULT_MAXSIZE;    //最大數量
        private int ttl = DEFAULT_TTL;        //过期时间（秒）

        public int getMaxSize() {
            return maxSize;
        }

        public int getTtl() {
            return ttl;
        }
    }

    /**
     * 创建基于Caffeine的Cache Manager
     *
     * @return
     */
    @Bean
    @Primary
    public CacheManager caffeineCacheManager() {
        SimpleCacheManager cacheManager = new SimpleCacheManager();

        ArrayList<CaffeineCache> caches = new ArrayList<CaffeineCache>();
        for (Caches c : Caches.values()) {
            caches.add(new CaffeineCache(c.name(),
                    Caffeine.newBuilder().recordStats()
                            .expireAfterWrite(c.getTtl(), TimeUnit.SECONDS)
                            .maximumSize(c.getMaxSize())
                            .build())
            );
        }

        cacheManager.setCaches(caches);

        return cacheManager;
    }

}
```



# Caffeine配置说明：

- initialCapacity=[integer]: 初始的缓存空间大小
- maximumSize=[long]: 缓存的最大条数
- maximumWeight=[long]: 缓存的最大权重
- expireAfterAccess=[duration]: 最后一次写入或访问后经过固定时间过期
- expireAfterWrite=[duration]: 最后一次写入后经过固定时间过期
- refreshAfterWrite=[duration]: 创建缓存或者最近一次更新缓存后经过固定的时间间隔，刷新缓存
- weakKeys: 打开key的弱引用
- weakValues：打开value的弱引用
- softValues：打开value的软引用
- recordStats：开发统计功能