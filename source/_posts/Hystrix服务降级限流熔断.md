---
title: Hystrix服务降级限流熔断
date: 2020-05-09 15:19:19
tags: [Hystrix,SpringCloud]
---

# Hystrix服务降级限流熔断

## 降级

### 主启动类激活

```java
添加注解
   @EnableCircuitBreaker
```

### 使用的两种方式

```java
1.注解 :  @HystrixCommand
2.硬编码  :extends HystrixCommand
```

## 使用

### 单方法降级配置

```java
@HystrixCommand(fallbackMethod="降级方法",commandProperties={
      //默认超时时间为1s 这边设置3秒
 @HystrixProperty(name="execution.isolation.thread.timeoutInMilliseconds",value="3000")
}) 
```

```java
1.fallbackMethod 降级的方法
 2. commandProperties 自定义的配置 比如超时时间的等
```

<!--more-->

###  全局降级默认配置

如果没有特殊配置fallbackMethod方法的注解 默认使用该降级方法

 ` @HystrixCommand`

```java
在类上加入
@DefaultProperties(defaultFallback="")
```

## 断路器 (服务熔断 )

需要手动开启断路器 还可以在 yml里面配置  

参数详情: HystrixCommandProperties.java

```java
@HystrixCommand(fallbackMethod="降级方法",commandProperties={
     //默认超时时间为1s 这边设置3秒
 @HystrixProperty(name="execution.isolation.thread.timeoutInMilliseconds",value="3000"),
    //是否开启断路器
 @HystrixProperty(name="ciruitBreaker.enabled",value="true"),
    //请求次数 默认20次
 @HystrixProperty(name="ciruitBreaker.requestVolumeThreshold",value="10"),
   //时间窗口期 默认
 @HystrixProperty(name="ciruitBreaker.sleepWindowInMilliseconds",value="10000"),
  //失败率达到多少后跳闸 默认50%
 @HystrixProperty(name="ciruitBreaker.errprThresholdPercentage",value="60")
}) 
```

## 限流

###  第一种 根据线程数

```java
  @HystrixCommand(
            commandKey = "helloCommand",//缺省为方法名
        threadPoolKey = "helloPool",//缺省为类名
        fallbackMethod = "fallbackMethod",//指定降级方法，在熔断和异常时会走降级方法
        commandProperties = {
        //超时时间
        @HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds", value = "1000")
},
        threadPoolProperties = {
                //并发，缺省为10
                @HystrixProperty(name = "coreSize", value = "5")
        }
    )
```



### 第二种 根据信号量

```java
#线程池的大小
Hystrix.threadpool.default.coreSize=1
#缓冲区大小，如果为-1则不缓冲，直接进行降级 fallback
hystrix.threadpool.default.maxQueueSize=200
#缓冲区的阈值，超限则直接降级
hystrix.threadpool.default.queueSizeRejectionThreshold=2

#执行策略
#资源隔离模式，默认thread，还有一种叫信号量
hystrix.command.default.execution.isolation.strategy=THREAD
#是否打开超时
hystrix.command.default.execution.timeout.enabled=true
#超时时间 默认1000毫秒
hystrix.command.default.execution.isolation.thread.timeoutInMilliseconds=2000
```



目的避免 某个服务占用大批量线程数 导致其他服务接收请求延时 从而导致整个服务请求量下降