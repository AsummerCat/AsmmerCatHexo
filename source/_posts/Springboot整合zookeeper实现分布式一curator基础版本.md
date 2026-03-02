---
title: springboot整合zookeeper实现分布式一curator基础版本
date: 2019-08-20 20:06:05
tags: [SpringBoot,zookeeper,分布式锁,curator]
---

# springboot整合zookeeper实现分布式一curator基础版本

# 这边实现了一个单Lock的demo

[demo地址](https://github.com/AsummerCat/zk-lock/tree/master/simple-zk-lock)

- idea 有集成一个zookeeper的插件 可以直接查看 zk服务器
- 或者选择下载一个 开源的ZooInspector.jar

# 首先需要了解一下 

zk分布式锁实现方式: 创建一个节点 利用临时节点来顺序下去  监听上一个节点 如果节点不见 获取当前锁

但是有一个问题 如果手动删除了临时节点 那么其他的监听器会继续执行下去 然后发现都是未获取锁 导致报错...

```
zookeeper原生客户端 (官方提供)
zkClient (开源的zk客户端 在原生api上封装 文档少)
Curator (netflix公司开源的客户端 apache顶级项目 更多时候用这个) 推荐
```

我们这边选用的是Curator

```
<dependency>
            <groupId>org.apache.curator</groupId>
            <artifactId>curator-recipes</artifactId>
            <version>4.2.0</version>
        </dependency>
```

<!--more-->

# 现在开始构建项目

这边啥都没配置 就只有一个lock

## 创建配置yml

```
server:
  port: 8090

spring:
  application:
    name: simple-zk-lock


#zk客户端配置重试次数
curator:
  retryCount: 5
  #重试间隔时间
  elapsedTimeMs: 5000
  # zookeeper 地址
  connectString: 127.0.0.1:2181
  # session超时时间
  sessionTimeoutMs: 60000
  # 连接超时时间
  connectionTimeoutMs: 5000
  lockPath: /lock

```



## 创建一个配置类 实例化  CuratorFramework

```
package com.linjingc.simplezklock.config;

import org.apache.curator.framework.CuratorFramework;
import org.apache.curator.framework.CuratorFrameworkFactory;
import org.apache.curator.retry.RetryNTimes;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * zk 初始化配置
 *
 * @author cxc
 * @date 2019年8月13日18:33:28
 */
@Configuration
public class CuratorConfiguration {

    @Value("${curator.retryCount}")
    private int retryCount;

    @Value("${curator.elapsedTimeMs}")
    private int elapsedTimeMs;

    @Value("${curator.connectString}")
    private String connectString;

    @Value("${curator.sessionTimeoutMs}")
    private int sessionTimeoutMs;

    @Value("${curator.connectionTimeoutMs}")
    private int connectionTimeoutMs;

    @Bean(initMethod = "start")
    public CuratorFramework curatorFramework() {
        return CuratorFrameworkFactory.newClient(
                connectString,
                sessionTimeoutMs,
                connectionTimeoutMs,
                new RetryNTimes(retryCount, elapsedTimeMs));
    }
}
```

## 锁方法 构建

## 这边就不继承java的Lock接口了

### 初始化

```
@Autowired
    private CuratorFramework zkClient;

    //这里后期需要修改
    //这边固定了一个key
    //需要注意的是 现在有个问题 因为是监听临时节点 如果节点被删除了 可能就会产生锁雪崩的情况
    @Value("${curator.lockPath}")
    private String lockPath;

//当前节点
    ThreadLocal<String> currentPath = new ThreadLocal<>();
//前节点
    ThreadLocal<String> beforePath = new ThreadLocal<>();
```



### 创建临时锁节点

```
 public void lock() {
        try {
            //根节点的初始化放在构造函数里面不生效
            if (zkClient.checkExists().forPath(lockPath) == null) {
                System.out.println("初始化根节点==========>" + lockPath);
                zkClient.create().creatingParentsIfNeeded().forPath(lockPath);
                System.out.println("当前线程" + Thread.currentThread().getName() + "初始化根节点" + lockPath);
            }
        } catch (Exception e) {
            throw new RuntimeException("构建根节点失败");
        }
        try {
            currentPath.set(zkClient.create().withMode(CreateMode.EPHEMERAL_SEQUENTIAL).forPath(lockPath + "/"));
        } catch (Exception e) {
            throw new RuntimeException("zk加锁失败");
        }
//检查是否获取成功锁 不成功阻塞线程
        checkLock();
    }
```

### 处理锁

```
  /**
     * 检查是否获取锁
     * 检查是否获取成功锁 不成功阻塞线程
     */
    private void checkLock() {
        //判断获取锁
        if (!tryLock()) {
            //监听
            waiForLock();
            //前锁解除继续判断获取
            checkLock();
        }
    }

```

### 获取锁

```
 //获取锁
      private boolean tryLock() {
        try {
            List<String> childrens = this.zkClient.getChildren().forPath(lockPath);
            Collections.sort(childrens);
            if (currentPath.get().equals(lockPath + "/" + childrens.get(0))) {
                System.out.println("当前线程获得锁" + currentPath.get());
                return true;
            } else {
                //取前一个节点
                int curIndex = childrens.indexOf(currentPath.get().substring(lockPath.length() + 1));
                //如果是-1表示children里面没有该节点
                beforePath.set(lockPath + "/" + childrens.get(curIndex - 1));
                System.out.println("前一个节点:" + lockPath + "/" + childrens.get(curIndex - 1));
            }
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
        return false;
    }
```



### 解锁      这边原本是准备在前面先判断下是否删除了前节点 然后发现首节点没有前节点

```
 public void unlock() {
        try {
                zkClient.delete().guaranteed().deletingChildrenIfNeeded().forPath(currentPath.get());
                System.out.println(currentPath.get() + "解锁成功");
                currentPath.remove();
                beforePath.remove();
        } catch (Exception e) {
            //若未删除成功，只要会话有效会在后台一直尝试删除
        }
    }

```

### 阻塞监听节点 等待锁处理   使用发令枪 监听到后继续执行 否则阻塞

```
/**
     * 阻塞监听节点  锁等待
     */
    private void waiForLock() {
        CountDownLatch cdl = new CountDownLatch(1);
        //创建监听器watch
        NodeCache nodeCache = new NodeCache(zkClient, beforePath.get());
        try {
            nodeCache.start(true);
            nodeCache.getListenable().addListener(new NodeCacheListener() {

                @Override
                public void nodeChanged() throws Exception {
                    System.out.println(nodeCache.getPath() + "节点监听事件触发");
                    cdl.countDown();
                }
            });
        } catch (Exception e) {
        }
        //如果前一个节点还存在，则阻塞自己
        try {
            if (zkClient.checkExists().forPath(beforePath.get()) != null) {
                cdl.await();
            }
        } catch (Exception e) {
        } finally {
            //阻塞结束，说明自己是最小的节点，则取消watch，开始获取锁
            try {
                nodeCache.close();
            } catch (IOException e) {
            }
        }
    }

```



# zk锁完整方法

```
package com.linjingc.simplezklock.zklock;

import org.apache.curator.framework.CuratorFramework;
import org.apache.curator.framework.recipes.cache.ChildData;
import org.apache.curator.framework.recipes.cache.NodeCache;
import org.apache.curator.framework.recipes.cache.NodeCacheListener;
import org.apache.zookeeper.CreateMode;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

import java.io.IOException;
import java.util.Collections;
import java.util.List;
import java.util.concurrent.CountDownLatch;

@Component("zklock")
public class ZKlock {

    @Autowired
    private CuratorFramework zkClient;

    //这里后期需要修改
    //这边固定了一个key
    //需要注意的是 现在有个问题 因为是监听临时节点 如果节点被删除了 可能就会产生锁雪崩的情况
    @Value("${curator.lockPath}")
    private String lockPath;

    ThreadLocal<String> currentPath = new ThreadLocal<>();
    ThreadLocal<String> beforePath = new ThreadLocal<>();


    private boolean tryLock() {
        try {
            List<String> childrens = this.zkClient.getChildren().forPath(lockPath);
            Collections.sort(childrens);
            if (currentPath.get().equals(lockPath + "/" + childrens.get(0))) {
                System.out.println("当前线程获得锁" + currentPath.get());
                return true;
            } else {
                //取前一个节点
                int curIndex = childrens.indexOf(currentPath.get().substring(lockPath.length() + 1));
                //如果是-1表示children里面没有该节点
                beforePath.set(lockPath + "/" + childrens.get(curIndex - 1));
                System.out.println("前一个节点:" + lockPath + "/" + childrens.get(curIndex - 1));
            }
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
        return false;
    }

    public void lock() {
        try {
            //根节点的初始化放在构造函数里面不生效
            if (zkClient.checkExists().forPath(lockPath) == null) {
                System.out.println("初始化根节点==========>" + lockPath);
                zkClient.create().creatingParentsIfNeeded().forPath(lockPath);
                System.out.println("当前线程" + Thread.currentThread().getName() + "初始化根节点" + lockPath);
            }
        } catch (Exception e) {
            throw new RuntimeException("构建根节点失败");
        }
        try {
            currentPath.set(zkClient.create().withMode(CreateMode.EPHEMERAL_SEQUENTIAL).forPath(lockPath + "/"));
        } catch (Exception e) {
            throw new RuntimeException("zk加锁失败");
        }

        checkLock();
    }


    /**
     * 检查是否获取锁
     */
    private void checkLock() {
        //判断获取锁
        if (!tryLock()) {
            //监听
            waiForLock();
            //前锁解除继续判断获取
            checkLock();
        }
    }


    public void unlock() {
        try {
                zkClient.delete().guaranteed().deletingChildrenIfNeeded().forPath(currentPath.get());
                System.out.println(currentPath.get() + "解锁成功");
                currentPath.remove();
                beforePath.remove();
        } catch (Exception e) {
            //guaranteed()保障机制，若未删除成功，只要会话有效会在后台一直尝试删除
        }
    }

    private void waiForLock() {
        CountDownLatch cdl = new CountDownLatch(1);
        //创建监听器watch
        NodeCache nodeCache = new NodeCache(zkClient, beforePath.get());
        try {
            nodeCache.start(true);
            nodeCache.getListenable().addListener(new NodeCacheListener() {

                @Override
                public void nodeChanged() throws Exception {
                    System.out.println(nodeCache.getPath() + "节点监听事件触发");
                    cdl.countDown();
                }
            });
        } catch (Exception e) {
        }
        //如果前一个节点还存在，则阻塞自己
        try {
            if (zkClient.checkExists().forPath(beforePath.get()) != null) {
                cdl.await();
            }
        } catch (Exception e) {
        } finally {
            //阻塞结束，说明自己是最小的节点，则取消watch，开始获取锁
            try {
                nodeCache.close();
            } catch (IOException e) {
            }
        }
    }


}

```

