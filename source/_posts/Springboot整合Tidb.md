---
title: SpringBoot整合Tidb
date: 2024-06-24 22:59:42
tags: [SpringBoot,Tidb]
---
## 依赖导入

```
就是普通的mysql的驱动之类的 tidb高度匹配mysql

<!-- Spring Boot Starter Parent -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.6.x</version>
</parent>

<!-- 添加 MyBatis -->
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>2.2.x</version>
</dependency>

<!-- 添加 Spring Data JPA -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>

<!-- MySQL 数据库驱动 -->
<dependency>
    <groupId>mysql</mysql>
    <artifactId>mysql-connector-java</artifactId>
    <version>8.0.x</version>
</dependency>

```

