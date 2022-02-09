---
title: Springcloud整合Zipkin和sleuth微服务链路跟踪
date: 2019-08-03 11:50:23
tags: [SpringBoot,springCloud,Zipkin,sleuth,链路跟踪]
---

# Springcloud整合Zipkin和sleuth微服务链路跟踪

[demo地址](https://github.com/AsummerCat/sleuth-demo)

# 创建Zipkin服务器 

需要注意的是2.0之后 不支持自定义创建@EnableZipkinStreamServer

https://github.com/openzipkin/zipkin/issues/1962 

官方有提供一个jar

官方文档: https://zipkin.io/pages/quickstart.html 

需要注意的是zipkin下载后maven编译下载包比较多

```java
无论您如何启动Zipkin，请浏览http：// your_host：9411以查找跟踪！
```

如果用 Docker 的话，直接

```
docker run -d -p 9411:9411 openzipkin/zipkin
```

# 创建eureka注册中心

略

<!--more-->

# 创建两个服务 A B

## 创建服务A

### 导入pom

```java
        <!--开启zipkin服务链路跟踪 这个组合包 zipkin+sleuth-->
    <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-zipkin</artifactId>
    </dependency>
```

## 添加配置信息

```java
spring:
  application:
    name: server-a-demo

# 配置链路相关信息
  sleuth:
    sampler:
      #设置sleuth收集信息的比率为1,默认10%
      probability: 1
  zipkin:
    #zipkin服务端地址
    base-url: http://localhost:9411
    #discovery-client-enabled: true
```











## 创建服务B

### 导入pom

```java
        <!--开启zipkin服务链路跟踪 这个组合包 zipkin+sleuth-->
    <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-zipkin</artifactId>
    </dependency>
```

## 添加配置文件

```java
spring:
  application:
    name: server-b-demo

  # 配置链路相关信息
  sleuth:
    sampler:
      #设置sleuth收集信息的比率为1,默认10%
      probability: 1
  zipkin:
    #zipkin服务端地址
    base-url: http://localhost:9411
    #discovery-client-enabled: true
```





# 理解

```java
springcloud_ribbon_client1：

INFO [ribbon-client1,144f1891d3547500,144f1891d3547500,true] 1612 --- [nio-3333-exec-1] c.s.w.c.RibbonClient1Controller          : RibbonClient1Controller--test() is requesting....
springcloud_ribbon_client2：

 INFO [ribbon-client2,144f1891d3547500,e3822e407ad26f52,true] 5160 --- [nio-4444-exec-1] c.s.w.c.RibbonClient2Controller          : RibbonClient2Controller--callRabbitClient2 is requesting...
由此可以看到sleuth进行服务链路跟踪的一些重要元素，[ribbon-client2,144f1891d3547500,e3822e407ad26f52,true] 的含义如下：

【a】第一个参数ribbon-client2：应用名称，对应我们application.yml中定义的application-name。
【b】第二个参数144f1891d3547500：Trace ID, 标识一条请求链路，一条请求链路包含一个Trace ID，多个Span ID。一条链路上的Trace ID是相同的，注意上面的日志信息第二个参数即Trace ID是一样的。
【c】第三个参数e3822e407ad26f52：Span ID,一个基本的工作单元，如一个http请求。
【d】第四个参数true：表示是否要将该信息输出到Zipkin等服务中来收集和展示。
```



