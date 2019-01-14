---
title: Hystrix断路器服务降级
date: 2019-01-13 15:27:59
tags: [SpringCloud,Hystrix]
---

# [demo地址](https://github.com/AsummerCat/hystrix)

# Hystrix断路器

这个是用来 比如A服务超时 或者挂了 可以服务降级处理

如果当前服务 超时很多次 比如 10次错误8次 服务自动降级  不会再去等待->超时->降级  

Hystrix具备了服务降级、服务熔断、线程隔离、请求缓存、请求合并以及服务监控等强大功能。

<!--more-->

# 首先导入相关pom文件

```java
<!-- spring-cloud-starter-hystrix依赖 -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-hystrix</artifactId>
</dependency>
```



# 启动类添加注解 两种方式

## 第一种

在应用主类中使用`@EnableCircuitBreaker`或`@EnableHystrix`注解开启Hystrix的使用

```java
package com.linjing.hystrixconsumer;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.circuitbreaker.EnableCircuitBreaker;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

@SpringBootApplication
@EnableDiscoveryClient
@EnableCircuitBreaker
public class HystrixConsumerApplication {

    public static void main(String[] args) {
        SpringApplication.run(HystrixConsumerApplication.class, args);
    }
}
```

 

## 第二种

**这里我们还可以使用Spring Cloud应用中的@SpringCloudApplication注解来修饰应用主类，该注解的具体定义如下所示。我们可以看到该注解中包含了上我们所引用的三个注解，这也意味着一个Spring Cloud标准应用应包含服务发现以及断路器。**

```java
`   @Target({ElementType.TYPE})
    @Retention(RetentionPolicy.RUNTIME)
    @Documented
    @Inherited
    @SpringBootApplication
    @EnableDiscoveryClient
    @EnableCircuitBreakerpublic 
    @interface SpringCloudApplication {}`
```

直接可以使用

```JAVA
package com.linjing.hystrixconsumer;

import org.springframework.boot.SpringApplication;
import org.springframework.cloud.client.SpringCloudApplication;

@SpringCloudApplication
public class HystrixConsumerApplication {

    public static void main(String[] args) {
        SpringApplication.run(HystrixConsumerApplication.class, args);
    }
}

```



# 添加 客户端的消费者方式 Feign

## 添加pom文件

```java
<dependency>
   <groupId>org.springframework.cloud</groupId>
   <artifactId>spring-cloud-starter-feign</artifactId>
</dependency>
```

## 接着在启动类上加入 开启Feign的注解

```java
修改应用主类。通过**@EnableFeignClients**注解开启扫描Spring Cloud Feign客户端的功能

```

这样一个基本的项目就完成的

# 配置文件部分

```java
spring:
  application:
    name: eureka-client
server:
  port: 8084

management:
  security:
    enabled: false
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8080/eureka/

## 开启feign默认开启的hystrix
feign:
  hystrix:
    enabled: true

ribbon:
  MaxAutoRetries: 1 #最大重试次数，当Eureka中可以找到服务，但是服务连不上时将会重试
  MaxAutoRetriesNextServer: 1 #切换实例的重试次数
  OkToRetryOnAllOperations: false # 对所有的操作请求都进行重试，如果是get则可以，如果是post,put等操作没有实现幂等的情况下是很危险的，所以设置为false
  ConnectTimeout: 1000 #请求连接的超时时间
  ReadTimeout: 1800 #请求处理的超时时间

#设置hystrix的超时时间 用来配置调用到副逻辑
hystrix:
  command:
    default:
      execution:
        timeout:
          enabled: true
        isolation:
          thread:
            timeoutInMilliseconds: 30000
            #设置回退的最大线程数
          semaphore:
            maxConcurrentRequests: 100
            #如果配置ribbon的重试，hystrix的超时时间要大于ribbon的超时时间，ribbon才会重试
            #hystrix的超时时间=(1 + MaxAutoRetries + MaxAutoRetriesNextServer) * ReadTimeout 比较好，具体看需求

```



# 剩下客户端的controller和server

略

# 现在配置服务调用失败的处理



## 在配置文件中开启Feign的Hystrix

```java
## 开启feign默认开启的hystrix
feign:
  hystrix:
    enabled: true
```

如果不开启的话 到需要降级的时候会报

```
Whitelabel Error Page
This application has no explicit mapping for /error, so you are seeing this as a fallback.

Sun Jan 13 18:04:35 CST 2019
There was an unexpected error (type=Internal Server Error, status=500).
index short-circuited and fallback failed.
```

服务降级

**这个需要在调用地方处理 别写在controller**

## 在调用的地方写入一个注解

在为具体执行逻辑的函数上增加`@HystrixCommand`注解来指定服务降级方法

```java
@HystrixCommand(fallbackMethod = "fallback")
```

---

如果是使用 Feign 可以加上

*// @FeignClient注解添加fallback属性* 

例如:

### feign调用类

```
package com.linjing.hystrixconsumer.server;

import org.springframework.cloud.netflix.feign.FeignClient;
import org.springframework.stereotype.Component;
import org.springframework.web.bind.annotation.RequestMapping;

/**
 * @author cxc
 * @date 2019/1/13 16:31
 */
//服务名称 注解只能放在类上
@Component
@FeignClient(value = "Testing-services", fallback = TestServiceFallback.class)
public interface TestServer {

    @RequestMapping("test")
    String index();
}
```

### 服务降级类

降级类 实现 feign接口

```java
package com.linjing.hystrixconsumer.server;

import org.springframework.stereotype.Component;


@Component
class TestServiceFallback implements TestServer {
    @Override
    public String index() {
        return "请稍后再试~~~";
    }
}
```

### 客户端调用类

```java
package com.linjing.hystrixconsumer.server;

import com.netflix.hystrix.contrib.javanica.annotation.HystrixCommand;
import com.netflix.hystrix.contrib.javanica.annotation.HystrixProperty;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

/**
 * @author cxc
 * @date 2019/1/13 16:31
 */
@Service
public class TestServerImpl {
    @Autowired
    private TestServer testServer;

    @HystrixCommand(commandProperties = {
            //设置熔断
            @HystrixProperty(name = "circuitBreaker.enabled", value = "true"),
            //时间滚动中最小请求参数，只有在一个统计窗口内处理的请求数量达到这个阈值，才会进行熔断与否的判断
            @HystrixProperty(name = "circuitBreaker.requestVolumeThreshold", value = "10"),
            //休眠时间窗
            @HystrixProperty(name = "circuitBreaker.sleepWindowInMilliseconds", value = "20000"),
            //错误百分比，判断熔断的阈值，默认值50，表示在一个统计窗口内有50%的请求处理失败，会触发熔断
            @HystrixProperty(name = "circuitBreaker.errorThresholdPercentage", value = "40")})
    public String index() {
        return testServer.index();
    }
}
```

 如果在一定时间内调用服务失败 会降级处理



# 需要注意

如果如果 hystrix的超时时间 会降级处理 所以请求时间要小于hystrix超时时间

# 完成