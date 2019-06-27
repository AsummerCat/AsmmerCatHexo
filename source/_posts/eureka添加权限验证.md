---
title: eureka添加权限验证
date: 2019-06-27 21:40:26
tags: [springCloud,eureka]
---

# eureka添加权限验证

版本号 <spring-cloud.version>Greenwich.SR1</spring-cloud.version>

## 导入pom文件

```java
  <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-security</artifactId>
        </dependency>
```

## 开启权限认证

<!--more-->

```java
   # 添加密码校验
 spring:
   security:
     user:
       name: 13003808707
       password: pass
       
       
       如果版本不同需要添加
       security.basic.enabled=true
```

这样就简单的配置了密码  需要密码才可以登录eureka

## 高版本的security中默认开启csrf 需要手动关闭

```java
package com.linjingc.eurekaserver.config;

import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;

@EnableWebSecurity
@Configuration
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.csrf().disable(); //关闭csrf
        http.authorizeRequests().anyRequest().authenticated().and().httpBasic(); //开启认证
    }

}
```

不然可能导致

```java
com.netflix.discovery.shared.transport.TransportException: Cannot execute request on any known server
无法注册服务
```



# 客户端操作

```java
配置文件中写入

eureka:
  client:
    serviceUrl:
      defaultZone:  http://<username>:<password>@localhost:8761/eureka
```

