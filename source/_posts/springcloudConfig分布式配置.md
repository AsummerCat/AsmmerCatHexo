---
title: springcloudConfig分布式配置
date: 2020-05-09 14:56:16
tags: [springCloud,SpringCloudConfig]
---

# springcloudConfig分布式配置

## 服务端的使用

```java
启动类加入@EnableConfigServer
```

### pom 加入config server

```java
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-config-server</artifactId>
</dependency>
```

###  application.yml里面加入配置

<!--more-->

```java
spring:
  application:
    name: ConfigServer
  cloud:
    config:
      server:
        git:
          uri: https://github.com/AsummerCat/SpringCloudConfigFileTest
          # 这里是配置git下面的具体目录 会    -> uri地址下的config-server目录中
          # {application} 自动根据项目名称去匹配
         # search-paths: config-client
          search-paths: {application}
```

## 客户端的使用

### pom加入 config client

```java
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-config</artifactId>
</dependency>
```

### 创建bootstrap.yml

```java
spring:
  application:
    name: configClient
  cloud:
    config:
      profile: pro
      # 分支
      label: master
      overrideNone: false
      discovery:
        enabled: true
        service-id: CONFIGSERVER

```

## 手动刷新

需要整合actuator 手动刷新资源

```java
在类上添加
@RefreshScope
所以这段话的重点就是:
所有@RefreshScope的Bean都是延迟加载的，只有在第一次访问时才会初始化
刷新Bean也是同理，下次访问时会创建一个新的对象
它在调用refresh方法的时候,会去调用工厂摧毁已生成的bean对象
也就是SpringCloud暴露了一个接口 /refresh 来给我们去刷新配置,但是SpringCloud 2.0.0以后,有了改变.
我们需要在bootstrap.yml里面加上需要暴露出来的地址
management:
  endpoints:
    web:
      exposure:
        include: refresh,health
        
现在的地址也不是/refresh了,而是/actuator/refresh
需要注意的是请求需要是Post请求
这边我们可以手动添加一个内存角色刷新资源 或者关闭权限信息 当然我们这边只是测试
在配置文件中加入 
management:
  security:
    enabled: false
    # 这里表示关闭权限信息
```

## 可以整合消息总线 BUS 实现自动刷新所有节点

### 导入bus的pom

```java
dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-bus-amqp</artifactId>
</dependency>
```

### 修改application.yml

```java
加入rabbitMQ的配置
spring:
  rabbitmq: # 现在将集成RabbitMQ作为消息服务总线处理
    host: 112.74.43.136  # RabbitMQ主机服务地址
    port: 5672 # RabbitMQ的监听端口
    username: cat # 用户名
    password: cat # 密码
    
management:
  endpoints:
    web:
      exposure:
          # 暴露bus刷新节点
        include: 'bus-refresh'    
```

启动此微服务信息，而后一定要保证该微服务已经注册到了 Eureka 之中，同时还需要观察出现 的访问路径，此时会出现有一个重要的路径信息：

```java
restartedMain] o.s.b.a.e.mvc.EndpointHandlerMapping : Mapped "{[/bus/refresh],methods=[POST]}"
onto public void org.springframework.cloud.bus.endpoint.RefreshBusEndpoint.refresh(java.lang.String)
```

我们可以在控制台中看到如下内容，在启动时候，客户端程序多了一个/bus/refresh请求。

```java
bus/refresh?destination=服务名:9000 局部刷新
/bus/refresh 全局刷新
```

