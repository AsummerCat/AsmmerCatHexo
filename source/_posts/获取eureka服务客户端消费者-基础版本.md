---
title: 获取eureka服务客户端消费者-基础版本
date: 2019-01-10 09:41:21
tags: [SpringCloud,eureka]
---

# [demo地址](https://github.com/AsummerCat/springCloudCustomer)

# eureka注册中心 (略)

这个操作简单已经注册成功了  主要是 消费端的使用

## eureka服务端 (略)

这个就是注册个服务到eureka就可以了

<!--more-->

## 使用LoadBalancerClient

在Spring Cloud Commons中提供了大量的与服务治理相关的抽象接口，包括`DiscoveryClient`、这里我们即将介绍的`LoadBalancerClient`等。Spring Cloud做这一层抽象，很好的解耦了服务治理体系，使得我们可以轻易的替换不同的服务治理设施。

从`LoadBalancerClient`接口的命名中，我们就知道这是一个负载均衡客户端的抽象定义，下面我们就看看如何使用Spring Cloud提供的负载均衡器客户端接口来实现服务的消费。



## 引入相关pom文件

```java
 <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-eureka</artifactId>
            <version>1.3.5.RELEASE</version>
        </dependency>
        <!-- 健康监控-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
    </dependencies>
```

## 初始化RestTemplate

- 创建应用主类。初始化`RestTemplate`，用来真正发起REST请求。`@EnableDiscoveryClient`注解用来将当前应用加入到服务治理体系中。

- ```java
  @Bean
  	public RestTemplate restTemplate() {
  		return new RestTemplate();
  	}
  ```



## 创建一个接口用来消费eureka-client提供的接口

```java
@RestController
public class DcController {

    @Autowired
    LoadBalancerClient loadBalancerClient;
    @Autowired
    RestTemplate restTemplate;

    @GetMapping("/consumer")
    public String dc() {
        ServiceInstance serviceInstance = loadBalancerClient.choose("eureka-client");
        String url = "http://" + serviceInstance.getHost() + ":" + serviceInstance.getPort() + "/dc";
        System.out.println(url);
        return restTemplate.getForObject(url, String.class);
    }
}
```



这样直接访问 请求就回通过loadBalancerClient获取可用的服务 调用端口号 发送请求