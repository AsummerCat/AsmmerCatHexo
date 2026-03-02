---
title: SpringBoot整合dubbo基础版本一
date: 2019-09-26 14:47:31
tags: [SpringBoot,dubbo]
---

# SpringBoot整合dubbo(一)基础版本

[demo地址](https://github.com/AsummerCat/dubbo-demo)

分三个模块

- 消费者
- 生产者
- api   这边放接口引入
- 如果后期还有实体的话 创建一个 entity的模块

# 首先还是导入pom

```
     <!-- https://mvnrepository.com/artifact/org.apache.dubbo/dubbo-spring-boot-starter -->
        <dependency>
            <groupId>org.apache.dubbo</groupId>
            <artifactId>dubbo-spring-boot-starter</artifactId>
            <version>2.7.3</version>
        </dependency>

        <dependency>
            <groupId>org.apache.curator</groupId>
            <artifactId>curator-recipes</artifactId>
            <version>4.2.0</version>
        </dependency>
```

<!--more-->

# 项目结构

![结构](/img/2019-09-16/1.png)

api中存放接口及其对应的mock类

consumer存放消费者 

producer存放生产者

# 开始搭建

## 创建api-demo

就把上面的pom内容复制进去

其他地方引用api-demo 所以也就不用导入重复的pom

![api-demo的pom](/img/2019-09-16/2.png)

### 创建接口

```
package com.linjingc.apidemo.service;

public interface SayService {

    public String sayHello();
}
```

### 及其对应的mock (降级类)

```
package com.linjingc.apidemo.service;

public class SayServiceMock implements SayService {
    public SayServiceMock() {
    }

    @Override
    public String sayHello() {
        return "请求失败";
    }
}

```

![service内容](/img/2019-09-16/3.png)

# 生产者的创建

- 消费者和生产者 只要导入api的模块就可以了

## pom.xml

![导入api-demo](/img/2019-09-16/4.png)

需要注意的是 因为是父子项目 模块需要先打包

## dubbo项目配置

```
server:
  port: 8085 #Tomcat端口
dubbo:
  application:
    name: producer-demo #应用名
    qos-accept-foreign-ip: false
    qos-port: 33333
    qos-enable: true
  registry:
    address: zookeeper://127.0.0.1:2181 #zookeeper地址
  protocol:
    name: dubbo
    port: 20890 #dubbo服务暴露的端口
  scan:
    base-packages: ba com.linjingc.producerdemo.service #扫描的包名
```

qos 是开启检测的内容 

这个就不介绍了

## 在生产者启动类添加注解

```
@EnableDubbo
```

标记启动dubbo

### 编写dubbo的接口实现类

需要添加`@Service`注解

需要注意的是 这个注解是

```
import org.apache.dubbo.config.annotation.Service;
```

这个包下的

### 具体接口案例

```
package com.linjingc.producerdemo.service;

import com.linjingc.apidemo.service.SayService;
import org.apache.dubbo.config.annotation.Service;

@Service(timeout = 3000,deprecated=true,accesslog="true",version = "1.0")
public class Say1ServiceImpl implements SayService {
    @Override
    public String sayHello() {
        return "再见了我的朋友2";
    }
}

```



# 消费者的构建

导入pom和配置文件跟生产者一样

## 在生产者启动类添加注解

```
@EnableDubbo
```

标记启动dubbo

### 使用生产者接口

就跟引入`@Autowired`一样的

 使用`Reference`注解

```
import org.apache.dubbo.config.annotation.Reference;

```

都是dubbo包下的

### 具体使用案例

```
package com.linjingc.consumerdemo.service;

import com.linjingc.apidemo.service.SayService;
import com.linjingc.apidemo.service.TestService;
import org.apache.dubbo.config.annotation.Reference;
import org.springframework.stereotype.Service;

@Service
public class HelloService {
    @Reference
    private TestService testService;
    @Reference(group="hello",mock = "true",id="sayServiceImpl")
    private SayService sayService;
    @Reference(version = "1.0")
    private SayService sayService1;

    public String hello() {
        return testService.test();
    }

    public String sayHello() {
          System.out.println( sayService.sayHello());
          System.out.println( sayService1.sayHello());
        return sayService1.sayHello();
    }
}

```

 这样基本就构建成功了

