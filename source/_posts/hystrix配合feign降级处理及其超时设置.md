---
title: hystrix配合feign降级处理及其超时设置
date: 2019-06-27 21:36:59
tags: [springCloud,Hystrix,feign]
---

# hystrix配合feign降级处理及其超时设置

## 导入pom

```java
 <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>
```

<!--more-->

## 启用 feign

```java
在启动类上添加
//开启Feign
@EnableFeignClients
//这个是组合注解
@SpringCloudApplication

```

## 配置feign调用类

```java
@Service
@FeignClient(value = "Tip-Producer-Server", fallbackFactory = TipsServerFallBackFactory.class)
public interface TipsService {

    @RequestMapping(value = "sendTip/{tip}")
    String sendTip(@PathVariable("tip") String tip);

    @RequestMapping(value = "index")
    String index();
}
```

### FeignClient 详解

```java
value  连接的服务名
url    调试时候指定的服务器
fallbackFactory 降级方法(可以写入异常信息)
fallback        降级方法
```

### fallbackFactory的实现

```java
@Component
public class TipsServerFallBackFactory implements FallbackFactory<TipsService> {
    private static final Logger logger = LoggerFactory.getLogger(TipsServerFallBackFactory.class);

    @Override
    public TipsService create(Throwable throwable) {
        logger.info("fallback reason was: {} ", throwable.getMessage());
        return new TipsService() {
            @Override
            public String sendTip(String tip) {
                return "sendTip请求失败";
            }

            @Override
            public String index() {
                return "index请求失败";
            }
        };

    }
}

```



## fallback的实现

```java
@Component
public class TipsServiceFallBack implements TipsService {

    @Override
    public String sendTip(String tip) {
        return "链接失败";
    }

    @Override
    public String index() {
        return "查询首页失败";
    }
}

```

## 配置文件的修改

### 基础信息设置

```java
spring:
  application:
    name: Tip-Consumer
server:
  port: 8300
      
 ## eureka 客户端基本配置
eureka:
  instance:
    prefer-ip-address: true
    instance-id: ${spring.cloud.client.ip-address}:${server.port}
  client:
    serviceUrl:
      defaultZone: http://admin:pass@localhost:8100/eureka     

```

### 开启feign的hystrix

```java
## 开启feign默认开启的hystrix 不开启会导致降级失败
feign:
  hystrix:
    enabled: true
```

2.0版本以上 默认是关闭feign的hystrix的

### 开启Ribbon的重试机制 

feign的重试机制为5次  修改为Ribbon好控制

```java
ribbon:
  MaxAutoRetries: 1 #同一台实例最大重试次数,不包括首次调用
  MaxAutoRetriesNextServer: 1 #重试负载均衡其他的实例最大重试次数,不包括首次调用
  OkToRetryOnAllOperations: false # 对所有的操作请求都进行重试，如果是get则可以，如果是post,put等操作没有实现幂等的情况下是很危险的，所以设置为false
  ConnectTimeout: 1000 #请求连接的超时时间
  ReadTimeout: 5000 #请求处理的超时时间
```

```java
# 根据上面的参数计算重试的次数：#MaxAutoRetries+MaxAutoRetriesNextServer+(MaxAutoRetries *MaxAutoRetriesNextServer) 即重试3次 则一共产生4次调用
```



- 如果你需要开启Feign的重试机制的话

- ```java
  一般情况下 都是 ribbon 的超时时间（<）hystrix的超时时间（因为涉及到ribbon的重试机制） 
  因为ribbon的重试机制和Feign的重试机制有冲突，所以源码中默认关闭Feign的重试机制
  
  要开启Feign的重试机制如下：（Feign默认重试五次 源码中有）
  @Bean
  Retryer feignRetryer() {
          return  new Retryer.Default();
  }
  如果不配置ribbon的重试次数，默认会重试一次 
  注意： 
  默认情况下,GET方式请求无论是连接异常还是读取异常,都会进行重试 
  非GET方式请求,只有连接异常时,才会进行重试
  ```

## 开启断路器的配置

```java
hystrix:
  # 核心线程池大小  默认10
  coreSize: 20
  # 最大最大线程池大小
  maximumSize: 30
  # 此属性允许maximumSize的配置生效。 那么该值可以等于或高于coreSize。 设置coreSize <maximumSize会创建一个线程池，该线程池可以支持maximumSize并发，但在相对不活动期间将向系统返回线程。 （以keepAliveTimeInMinutes为准）
  allowMaximumSizeToDivergeFromCoreSize: true
  # 请求等待队列
  maxQueueSize: 10
  # 队列大小拒绝阀值 在还未超过请求等待队列时也会拒绝的大小
  queueSizeRejectionThreshold: 10
```

以上是设置一些基础信息

#### 默认的断路配置

```java
 command:
    default:
      execution:
        timeout:
          #如果enabled设置为false，则请求超时交给ribbon控制,为true,则超时作为熔断根据
          enabled: true
        isolation:
          thread:
            timeoutInMilliseconds: 1000 #断路器超时时间，默认1000ms
            #设置回退的最大线程数
          semaphore:
            maxConcurrentRequests: 100
              #如果配置ribbon的重试，hystrix的超时时间要大于ribbon的超时时间，ribbon才会重试
              #hystrix的超时时间=(1 + MaxAutoRetries + MaxAutoRetriesNextServer) * ReadTimeout 比较好，具体看需求
              #isolation:
              #隔离策略，有THREAD和SEMAPHORE
            #THREAD - 它在单独的线程上执行，并发请求受线程池中的线程数量的限制
            #SEMAPHORE - 它在调用线程上执行，并发请求受到信号量计数的限制
```

#### 为单独的方法设置断路器配置 和超时时间

```java
  ##单独方法(方法名#方法(类型)) 添加 断路器设置
    TipsService#sendTip(String):
      execution:
        timeout:
          enabled: true
        isolation:
          thread:
            timeoutInMilliseconds: 22000
    TipsService#index():
      execution:
        timeout:
          enabled: true
        isolation:
          thread:
            timeoutInMilliseconds: 18000
```

需要注意的是

```java
command.execution.timeout.enabled=true
          #如果enabled设置为false，则请求超时交给ribbon控制,为true,则超时作为熔断根据
```

### 关于超时信息的一些注意内容

```
  #MaxAutoRetries+MaxAutoRetriesNextServer+(MaxAutoRetries *MaxAutoRetriesNextServer) 即重试3次 则一共产生4次调用
  
  在请求调用的时候 ribbon的总请求时间需要 hystrix设置的超时时间
  最大总请求时间MaxAutoRetries+MaxAutoRetriesNextServer+(MaxAutoRetries *MaxAutoRetriesNextServer)*ReadTimeout
  如果超过这个时间就会被降级
```





# 完整配置文件

```java
spring:
  application:
    name: Tip-Consumer
server:
  port: 8300


## eureka 客户端基本配置
eureka:
  instance:
    prefer-ip-address: true
    instance-id: ${spring.cloud.client.ip-address}:${server.port}
  client:
    serviceUrl:
      defaultZone: http://admin:pass@localhost:8100/eureka

## 开启feign默认开启的hystrix 不开启会导致降级失败
feign:
  hystrix:
    enabled: true

## 开启ribbon重试机制
ribbon:
  MaxAutoRetries: 1 #同一台实例最大重试次数,不包括首次调用
  MaxAutoRetriesNextServer: 1 #重试负载均衡其他的实例最大重试次数,不包括首次调用
  OkToRetryOnAllOperations: false # 对所有的操作请求都进行重试，如果是get则可以，如果是post,put等操作没有实现幂等的情况下是很危险的，所以设置为false
  ConnectTimeout: 1000 #请求连接的超时时间
  ReadTimeout: 5000 #请求处理的超时时间

  # 根据上面的参数计算重试的次数：
  #MaxAutoRetries+MaxAutoRetriesNextServer+(MaxAutoRetries *MaxAutoRetriesNextServer) 即重试3次 则一共产生4次调用


## 开启断路器
hystrix:
  # 核心线程池大小  默认10
  coreSize: 20
  # 最大最大线程池大小
  maximumSize: 30
  # 此属性允许maximumSize的配置生效。 那么该值可以等于或高于coreSize。 设置coreSize <maximumSize会创建一个线程池，该线程池可以支持maximumSize并发，但在相对不活动期间将向系统返回线程。 （以keepAliveTimeInMinutes为准）
  allowMaximumSizeToDivergeFromCoreSize: true
  # 请求等待队列
  maxQueueSize: 10
  # 队列大小拒绝阀值 在还未超过请求等待队列时也会拒绝的大小
  queueSizeRejectionThreshold: 10

  command:
    default:
      execution:
        timeout:
          #如果enabled设置为false，则请求超时交给ribbon控制,为true,则超时作为熔断根据
          enabled: true
        isolation:
          thread:
            timeoutInMilliseconds: 1000 #断路器超时时间，默认1000ms
            #设置回退的最大线程数
          semaphore:
            maxConcurrentRequests: 100
              #如果配置ribbon的重试，hystrix的超时时间要大于ribbon的超时时间，ribbon才会重试
              #hystrix的超时时间=(1 + MaxAutoRetries + MaxAutoRetriesNextServer) * ReadTimeout 比较好，具体看需求
              #isolation:
              #隔离策略，有THREAD和SEMAPHORE
            #THREAD - 它在单独的线程上执行，并发请求受线程池中的线程数量的限制
            #SEMAPHORE - 它在调用线程上执行，并发请求受到信号量计数的限制

    ##单独方法 添加 断路器设置
    TipsService#sendTip(String):
      execution:
        timeout:
          enabled: true
        isolation:
          thread:
            timeoutInMilliseconds: 22000
    TipsService#index():
      execution:
        timeout:
          enabled: true
        isolation:
          thread:
            timeoutInMilliseconds: 18000

```

