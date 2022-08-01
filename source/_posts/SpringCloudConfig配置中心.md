---
title: SpringCloudConfig配置中心
date: 2019-01-03 10:41:51
tags: [SpringCloud,SpringCloudConfig,分布式配置中心]
---

# [demo地址](https://github.com/AsummerCat/SpringCloudConfigTest)

# 使用版本 SpringBoot 1.5.3  SpringCloud Dalston.SR4

友情提示:

访问配置信息的URL与配置文件的映射关系如下：

- /{application}/{profile}[/{label}]
- /{application}-{profile}.yml
- /{label}/{application}-{profile}.yml
- /{application}-{profile}.properties
- /{label}/{application}-{profile}.properties

- spring.application.name：对应配置文件规则中的`{application}`部分
- spring.cloud.config.profile：对应配置文件规则中的`{profile}`部分
- spring.cloud.config.label：对应配置文件规则中的`{label}`部分
- spring.cloud.config.uri：配置中心`config-server`的地址

**这里需要格外注意：上面这些属性必须配置在bootstrap.properties中，这样config-server中的配置信息才能被正确加载。**

<!--more-->

## 简单版(单机) 

### 服务端

### 添加相应的pom文件

spring-cloud-config-server 服务端的依赖

```
	<dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-config-server</artifactId>
    </dependency>
```

### 在启动类上标明一个注解 表示启动config-server服务

```
package com.linjingc.SpringCloudconfigserver;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.config.server.EnableConfigServer;

/**
 * @author macpro
 */
@SpringBootApplication
//这边加上这注解 就代表启动了config server
@EnableConfigServer
public class SpringCloudConfigServerApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringCloudConfigServerApplication.class, args);
    }

}

```

## 在application.yml中添加配置信息

```
spring:
  application:
    name: config-server
  cloud:
    config:
      server:
          git:
            uri: https://github.com/AsummerCat/SpringCloudConfigFileTest
          # 这里是配置git下面的具体目录 会    -> uri地址下的config-server目录中
            search-paths: config-server
          #  password: 如果需要权限访问
          #  username: 如果需要权限访问
server:
  port: 8081

```



这样初步的服务端已经完成了

### 客户端

### 添加相关jar包信息

```
	<dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-config</artifactId>
            <version>1.2.2.RELEASE</version>
    </dependency>
        
         <!-- 健康监控-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
```

## 在bootstarp.yml 写入相关配置

这里因为虚拟机运行时  bootstarp 执行在application 之前

```
spring:
  application:
    name: config-client
  cloud:
    config:
    # 配置你需要链接的config-server地址
      uri: http://localhost:1201
      ## 配置文件名称
      profile: default
```

**这里需要注意的是 如果application.yml中配置了端口 如果config-server 联不上,会使用application中的port**

## 客户端获取信息

//按照上面的内容 一个简单的spring Cloud Config就完成了

下面看一下如何获取值

#### 配置文件

```
info:
  profile: dev
server:
  port: 8091
  
test: myCat
person:
  no1: 小明
  no2: 小东
```

需要注意的值 如果配置文件不存在这个key 应用去获取 会报错

#### 引用点

```
package com.linjingc.SpringCloudconfigclient.controller;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.cloud.context.config.annotation.RefreshScope;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RefreshScope
@RestController
class TestController {

    @Value("${test}")
    private String test;
    @Value("${person.no1}")
    private String to1;
    @Value("${person.no2}")
    private String to2;

    @RequestMapping("/from")
    public String from() {
        System.out.println(test);
        System.out.println(to1);
        System.out.println(to2);
        return this.test;
    }

}
```

**注意:这样获取值 如果服务器刷新值后 这边获取不到更新后的值**

两种方式解决  客户端手动刷新资源 或者服务端推送请求

客户端手动通过 /refresh 刷新资源

# actuator 手动刷新资源

这边加入了健康监控插件  在1.5版本里 如果你要刷新值

**@RefreshScope **

所以这段话的重点就是:

1. 所有@RefreshScope的Bean都是延迟加载的，只有在第一次访问时才会初始化
2. 刷新Bean也是同理，下次访问时会创建一个新的对象

它在调用refresh方法的时候,会去调用工厂摧毁已生成的bean对象

也就是SpringCloud暴露了一个接口 /refresh 来给我们去刷新配置,但是SpringCloud 2.0.0以后,有了改变.

我们需要在bootstrap.yml里面加上需要暴露出来的地址

```
management:
  endpoints:
    web:
      exposure:
        include: refresh,health
```

现在的地址也不是/refresh了,而是/actuator/refresh

需要注意的是请求需要是Post请求

另外需要设置一个密码才可以访问 不然没有权限

这边我们可以手动添加一个内存角色刷新资源   或者关闭权限信息 当然我们这边只是测试

```
在配置文件中加入 
management:
  security:
    enabled: false
    # 这里表示关闭权限信息
```

效果展示:

访问:http://localhost:8091/refresh

![效果图](/img/2019-1-3/configTest1.png)

![效果图](/img/2019-1-3/configTest.png)

# 加入eruka负载均衡

ConfigServer 和ConfigClient

## ConfigServer 

### 加入eruka的pom.xml

```
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-eureka</artifactId>
        </dependency>
```

  ### 启动类添加eureka注解

```
@EnableEurekaClient
```

### 接下来添加配置文件信息

```
在application.yml中加入
# 注册到eureka
eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8100/eureka/
```

使其注册到eureka上

这样configServer就注册上去了

## ConfigClient

跟ConfigServer  前两步一样 同样加入eureka的pom信息,在启动类加入注解  在注册到eureka

### 接下来修改配置文件信息 

修改原先直接调用config url 那部分的内容 使其去eureka寻找configServer

ootstrap.yaml中增加eureka连接配置，同时屏蔽`spring. cloud.config.uri` 增加：`spring.cloud.config.discovery.enabled`和`spring.cloud.config.discovery.serviceId`。

```
修改bootStrap.yml

spring:
  application:
    name: config-client
  cloud:
    config:
    # 配置你需要链接的config-server地址
    #(注释掉 指定的url) uri: http://user:password@localhost:8081
      ## 配置文件名称
      profile: dev
      discovery:
      # 表示使用服务发现组件中的Config Server，而不自己指定Config Server的uri，默认false
        enabled: true
        # 指定Config Server在服务发现中的serviceId，默认是configserver
        service-id: config-server
   #如果config-server需要权限验证的话     
      username: user
      password: password


# 注册到eureka
eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8100/eureka/
```

## 完成

这样就实现了 config的高可用 如果多个configSrever一起运行的时候,服务器挂掉一台 我们还是可以收到 配置信息

# 添加权限控制

这边权限控制的话 老样子 调用spring全家桶的security

## 首先导入相关pom文件

```
<dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-security</artifactId>
</dependency>
```

## 这边我们demo 也不做太多的权限 直接在内存中写入一个用户及其密码~~~~ 搞定

```
在配置文件中写入
#配置security的账号密码
security:
  basic:
    enabled: true
  user:
    name: user
    password: user
```

## 最后 修改客户端链接到config的地址  加入密码

格式:

```
  uri: http://localhost:8081 改为 uri: http://user:password@localhost:8081
```

# 加密/解密

## **使用前提**

在使用Spring Cloud Config的加密解密功能时，有一个必要的前提需要我们注意。为了启用该功能，我们需要在配置中心的运行环境中安装不限长度的JCE版本（Unlimited Strength Java Cryptography Extension）。虽然，JCE功能在JRE中自带，但是默认使用的是有长度限制的版本。我们可以从Oracle的官方网站中下载到它，它是一个压缩包，解压后可以看到下面三个文件：

```
README.txt
local_policy.jar
US_export_policy.jar
```

我们需要将local_policy.jar和US_export_policy.jar两个文件复制到$JAVA_HOME/jre/lib/security目录下，覆盖原来的默认内容。到这里，加密解密的准备工作就完成了。

![加密解密](/img/2019-1-3/加密test.png)

## **相关端点**

在完成了JCE的安装后，可以尝试启动配置中心。在控制台中，将会输出了一些配置中心特有的端点，主要包括：

- /encrypt/status：查看加密功能状态的端点
- /key：查看密钥的端点
- /encrypt：对请求的body内容进行加密的端点
- /decrypt：对请求的body内容进行解密的端点

可以尝试通过GET请求访问/encrypt/status端点，我们将得到如下内容：

```
{
 "description": "No key was installed for encryption service",
 "status": "NO_KEY"
}
```

该返回说明当前配置中心的加密功能还不能使用，因为没有为加密服务配置对应的密钥。

## **配置密钥**

我们可以通过encrypt.key属性在配置文件中直接指定密钥信息（对称性密钥），比如：

```
encrypt.key=CAT
```

加入上述配置信息后，重启配置中心，再访问/encrypt/status端点，我们将得到如下内容：

```
{
 "status": "OK"
}
```

此时，我们配置中心的加密解密功能就已经可以使用了，不妨尝试访问一下/encrypt和/decrypt端点来进行加密和解密的功能。注意，这两个端点都是POST请求，加密和解密信息需要通过请求体来发送。比如，以curl命令为例，我们可以通过下面的方式调用加密与解密端点：

```
$ curl localhost:7001/encrypt -d didispace
3c70a809bfa24ab88bcb5e1df51cb9e4dd4b8fec88301eb7a18177f1769c849ae9c9f29400c920480be2c99406ae28c7

$ curl localhost:7001/decrypt -d 3c70a809bfa24ab88bcb5e1df51cb9e4dd4b8fec88301eb7a18177f1769c849ae9c9f29400c920480be2c99406ae28c7
didispace
```

这里，我们通过配置encrypt.key参数来指定密钥的实现方式采用了对称性加密。这种方式实现比较简单，只需要配置一个参数即可。另外，我们也可以使用环境变量ENCRYPT_KEY来进行配置，让密钥信息外部化存储。

例子:

```
spring.datasource.username=cat
spring.datasource.password={cipher}dba6505baa81d78bd08799d8d4429de499bd4c2053c05f029e7cfbf143695f5b
```

在Spring Cloud Config中通过在属性值前使用`<font color"red">{cipher}</font>`前缀来标注该内容是一个加密值，当微服务客户端来加载配置时，配置中心会自动的为带有<font color"red">{cipher}</font>`前缀的值进行解密。通过该机制的实现，运维团队就可以放心的将线上信息的加密资源给到微服务团队，而不用担心这些敏感信息遭到泄露了。下面我们来具体介绍如何在配置中心使用该项功能。

# 高可用 动态刷新

动态刷新的两种方式

![](/img/2019-1-3/动态刷新two.png)