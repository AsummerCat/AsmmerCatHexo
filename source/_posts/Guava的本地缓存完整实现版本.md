---
title: Guava的本地缓存完整实现版本
date: 2019-06-01 14:03:08
tags: java
---

# Guava的本地缓存完整实现版本



## 创建一个策略接口

```java
package com.mdt.newdrugreview.utils.localcacheUtil;


/**
 * 类说明:策略接口，每个策略都必须实现这个标准的策略接口
 *
 * @author cxc
 * @date 2019年5月30日15:14
 */
public interface ILocalCache<K, V> {
    /**
     * 根据Key获取value
     *
     * @param k key
     * @return value
     */
    V get(K k);
}
```

<!--more-->

## 封装了对Guava Cache的利用，包括cache的创建、从数据源获取数据等

```
package com.mdt.newdrugreview.utils.localcacheUtil;

import org.spark_project.guava.cache.CacheBuilder;
import org.spark_project.guava.cache.CacheLoader;
import org.spark_project.guava.cache.LoadingCache;

import java.util.concurrent.ExecutionException;
import java.util.concurrent.TimeUnit;

/**
 * 策略辅助类
 * 封装了对Guava Cache的利用，包括cache的创建、从数据源获取数据等
 *
 * @param <K>
 * @param <V>
 */
public abstract class GuavaAbstractLoadingCache<K, V> {
    /**
     * 最大缓存条数
     */
    private int maximumSize = 1000;
    /**
     * 数据存在时长
     */
    private int expireAfterWriteDuration = 30;
    /**
     * 时间单位（分钟）
     */
    private TimeUnit timeUnit = TimeUnit.MINUTES;
    /**
     * 设置并发级别
     */
    private int levelNum = 8;


    private LoadingCache<K, V> cache;


    /**
     * 通过调用getCache().get(key)来获取数据
     *
     * @return cache
     */
    public LoadingCache<K, V> getCache() {
        //使用双重校验锁保证只有一个cache实例
        if (cache == null) {
            synchronized (this) {
                if (cache == null) {
                    //缓存数据的最大条目
                    cache = CacheBuilder.newBuilder().maximumSize(maximumSize)
                            //设置并发级别为8，并发级别是指可以同时写缓存的线程数 默认 4
                            .concurrencyLevel(8)
                            //数据被创建多久后被移除
                            .expireAfterWrite(expireAfterWriteDuration, timeUnit)
                            //启用统计   (用于缓存信息的统计 可选)
//							.recordStats()
                            .build(new CacheLoader<K, V>() {
                                @Override
                                public V load(K key) throws Exception {
                                    return fetchData(key);
                                }
                            });
                }
            }
        }

        return cache;
    }

    /**
     * 从缓存中获取数据（第一次自动调用fetchData从外部获取数据）
     *
     * @param key
     * @return Value
     * @throws ExecutionException
     */
    protected V getValue(K key) throws ExecutionException {
        V result = getCache().get(key);
        return result;
    }

    /**
     * 根据key从数据库或其他数据源中获取一个value，并被自动保存到缓存中。
     * 这个方法由子类实现，以达到不同策略从不同的数据源获取缓存信息的效果
     *
     * @param key
     * @return value, 连同key一起被加载到缓存中。
     */
    protected abstract V fetchData(K key);

}


```



## 具体实现类

```java
package com.mdt.newdrugreview.utils.localcacheUtil;

import org.jeecgframework.web.system.service.SystemService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.HashSet;

/**
 * 获取诊断信息同义词 缓存
 *
 * @param <String>
 * @param <List>
 * @author cxc
 * @date 2019年5月30日16:28:54
 */
@Component
public class GetCacheDiagnose<String, List> extends GuavaAbstractLoadingCache<String, List> implements ILocalCache<String, List> {

    @Autowired
    private SystemService systemService;

    @Override
    public List get(String key) {
        try {
            return getValue(key);
        } catch (Exception e) {
            return null;
        }
    }

    @Override
    @SuppressWarnings("unchecked")
    protected List fetchData(String key) {
        //当缓存中没有key的缓存时，就从这个方法来获取，一般地这里会从数据源取数据，这里模拟从数据库获取数据
        StringBuffer diagnose = new StringBuffer();
        java.lang.String[] split = key.toString().split(",");
        diagnose.append(split[0]);
        for (int i = 1; i < split.length; i++) {
            diagnose.append("','");
            diagnose.append(split[i]);
        }
        java.lang.String str = "('" + diagnose.toString() + "')";
        java.lang.String type = "'" + "诊断" + "'," + "'" + "ICD10" + "'," + "'" + "规则诊断" + "'";
        java.lang.String sql = "select p.source_name from p_base_datadictionary_mapp p where p.mapp_type in (" + type + ") and p.target_name in " + str + " and p.is_enable='1'";
        java.util.List<java.lang.String> list = new ArrayList<java.lang.String>(Arrays.asList(split));
        list.addAll(new ArrayList<java.lang.String>(systemService.<java.lang.String>findListbySql(sql)));
        //去重
        HashSet h = new HashSet(list);
        list.clear();
        list.addAll(h);

        return (List) list;
    }
}
```



## 测试方法

```
GetCacheDiagnose A=new GetCacheDiagnose();
A.get(key);
```

