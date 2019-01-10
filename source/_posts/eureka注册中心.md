---
title: eruka注册中心
date: 2019-01-05 19:41:55
tags: [SpringCloud,eureka]
---

# 利用eureka做注册中心 实现微服务的高可用

于Spring Cloud为服务治理做了一层抽象接口，所以在Spring Cloud应用中可以支持多种不同的服务治理框架，比如：Netflix Eureka、Consul、Zookeeper。在Spring Cloud服务治理抽象层的作用下，我们可以无缝地切换服务治理实现，并且不影响任何其他的服务注册、服务发现、服务调用等逻辑。

<!--more-->

# 服务端

## 导入pom.xml

```java
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-eureka-server</artifactId>
    </dependency>
</dependencies>
```



# 启动类上 开启注解

通过`@EnableEurekaServer`注解启动一个服务注册中心提供给其他应用进行对话。这一步非常的简单，只需要在一个普通的Spring Boot应用中添加这个注解就能开启此功能，

```java
package com.linjing.eureka;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;

@SpringBootApplication
//开启注册中心服务端
@EnableEurekaServer
public class EurekaApplication {

    public static void main(String[] args) {
        SpringApplication.run(EurekaApplication.class, args);
    }
}

```



## 修改配置文件

在默认设置下，该服务注册中心也会将自己作为客户端来尝试注册它自己，所以我们需要禁用它的客户端注册行为，只需要在`application.yml `配置文件中增加如下信息：

```java
spring:
  application:
    name: eureka-Server
server:
  port: 8100

eureka:
  instance:
  #设置当前实例的主机名称
    hostname: localhost
  client:
  #关闭本身的服务注册
    register-with-eureka: false
    #检索服务
    fetch-registry: false
    ##关闭保护模式
  server:
    enableSelfPreservation: false

```



启动  http://localhost:8100 

这样注册中心就启动了



# 创建“服务提供方”

## 导入pom.xml

```java
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-eureka</artifactId>
    </dependency>
</dependencies>
```

## 启动类中开启提供方注解

最后在应用主类中通过加上`@EnableDiscoveryClient`注解，该注解能激活Eureka中的DiscoveryClient实现，这样才能实现Controller中对服务信息的输出。

### 使用DiscoveryClient类

如果你的应用使用@EnableEurekaClient注解，那么只能使用eureka来发现服务实例。 
一个方法是使用com.netflix.discovery.DiscoveryClient

```java
package com.linjing.eurekaclient;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

@SpringBootApplication
//该注解能激活Eureka中的客户端使用
@EnableDiscoveryClient`
public class EurekaClientApplication {
    public static void main(String[] args) {
        SpringApplication.run(EurekaClientApplication.class, args);
    }
}
```

## 加入一个接口方法

其次，实现/test请求处理接口，通过DiscoveryClient对象，在日志中打印出服务实例的相关内容

```java
@RestController
public class TestController {

    @Autowired
    DiscoveryClient discoveryClient;

    @RequestMapping("test")
    public String info() {
        String services = "Services: " + discoveryClient.getServices();
        System.out.println(services);
        return services;
    }
}
```

## 修改配置文件内容application.yml

```java

spring:
  application:
  # 这里表示注册服务的名称
    name: eureka-client
server:
  port: 8201
eureka:
  client:
    serviceUrl:
    #需要注册的注册中心地址 要加上/eureka
    #如果服务注册中心为高可用集群时，多个注册中心地址以逗号分隔。
    #如果服务注册中心加入了安全验证，
    #这里配置的地址格式为： http://<username>:<password>@localhost:8761/eureka
    #其中 <username> 为安全校验的用户名；<password> 为该用户的密码
      defaultZone: http://localhost:8100/eureka

```

## 注册提供服务成功

![注册](/img/2019-1-5/eureka.png)

这里可以看到 我们注册了两个相同的服务



# 调用方



多种方式 (待更新)

