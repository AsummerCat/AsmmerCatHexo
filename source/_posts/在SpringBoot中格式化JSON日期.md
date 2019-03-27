---
title: 在SpringBoot中格式化JSON日期
date: 2019-03-27 21:06:12
tags: [SpringBoot]
---

# 在Spring Boot中格式化JSON日期

## Jackson格式化日期的各种方法

# 方式一:在日期字段上使用@JsonFormat

## 设置格式

```
//在需要的转换格式的字段上加入 这句话  
@JsonFormat(pattern="yyyy-MM-dd HH:mm:ss")  

```
## 设置时区

```
如果我们需要使用特定的时区，我们可以设置@JsonFormat的timezone属性  
@JsonFormat(pattern="yyyy-MM-dd HH:mm:ss",timezon="Europe/Zagreb")  

```
<!--more-->

# 方式二 全局修改配置文件

```
如果我们要为应用程序中的所有日期配置默认格式，则更灵活的方法是在application.properties中配置它：  
spring.jackson.date-format=yyyy-MM-dd HH:mm:ss
```

如果我们想要在json日期中使用特定时区 再加入

```

spring.jackson.time=Europe/Zagreb  

```
尽管设置这样的默认格式非常方便直接，但这种方法存在缺陷。不幸的是，它不适用于Java 8日期类型，如 LocalDate 和 LocalDateTime - 我们只能使用它来格式化java.util.Date或 java.util.Calendar类型的字段 。 但是，我们很快就会看到希望。
