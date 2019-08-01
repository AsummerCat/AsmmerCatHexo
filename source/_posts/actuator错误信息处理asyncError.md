---
title: actuator错误信息处理asyncError
date: 2019-08-01 22:27:16
tags: [actuator,springCloud,SpringBoot]
---

# 这个问题是在搭建spring-admin监控的时候发现的

版本如下

spring-boot: 2.1.2.RELEASE

spring-boot-admin:2.1.2

问题的相关描述看这里

https://github.com/spring-projects/spring-boot/issues/15057

里面有建议的暂行解决方案是降级tomcat

也可以使用jetty替换

去除tomcat的依赖

<!--more-->

如下

```
 <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <exclusions>
                <exclusion>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-starter-tomcat</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
```

添加jetty的依赖

```
<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jetty</artifactId>
        </dependency>
```


实测这个问题在jetty中不存在

另外还有一个关于spring-boot-admin的问题

如果server启动的时候没有client注册上,页面会一直显示加载中,查看请求的时候是application一直在请求