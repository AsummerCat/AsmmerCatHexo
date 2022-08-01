---
title: springboot整合h2内存数据库
date: 2019-07-16 17:09:43
tags: [SpringBoot,数据库]
---

# springboot整合h2内存数据库

[demo地址](https://github.com/AsummerCat/h2-demo)

# h2 

这个是一个开源的数据库 可以直接在内存中生出

直接上demo

# pom.xml 

```
  <dependency>
            <groupId>com.h2database</groupId>
            <artifactId>h2</artifactId>
            <scope>runtime</scope>
        </dependency>
```

<!--more-->

# 配置文件修改

```
server:
  port: 8090

spring:
  datasource:
    hikari:
      driver-class-name: org.h2.Driver
      jdbc-url: jdbc:h2:~/test
    platform: h2
    #初始化脚本
    schema: classpath:db/schema.sql
   
   #在浏览器中开启控制台
  h2:
    console:
      enabled: true
      settings:
        web-allow-others: true
      path: /h2
```

# 脚本

```
create table if not exists USER (
NAME varchar(100),
ID NUMBER,
AGE NUMBER(10),
PASSWORD VARCHAR(18));
```

然后正常启动就可以了

访问地址: ip:port/h2



