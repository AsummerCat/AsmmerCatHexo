---
title: SpringBootActuator端点
date: 2019-08-01 21:29:39
tags: [actuator,SpringCloud,SpringBoot]
---

# **SpringBoot actuator各个端点报告及说明**

[参考地址](https://blog.csdn.net/IT_faquir/article/details/80465725)

## [官网文档 获版本.16的奔波最新端口详情](https://docs.spring.io/spring-boot/docs/2.1.6.RELEASE/reference/htmlsingle/#production-ready)

## 端点详情 (1.5的版本)

![端点](/img/2019-08-01/actuator端点.png)

<!--more-->



## 端口详情 (2.16版本)

* auditevents       公开当前应用程序的审核事件信息。

* beans               显示应用程序中所有Spring bean的完整列表。

* caches             暴露可用的缓存。
* conditions        显示在配置和自动配置类上评估的条件以及它们匹配或不匹配的原因。  
* configprops     显示所有的整理列表`@ConfigurationProperties`。
* env                  露出Spring的属性`ConfigurableEnvironment`。 上下文bean 注册的bean
* flyway             显示已应用的任何Flyway数据库迁移。
* health(默认开启)             显示应用健康信息。
* httptrace              显示HTTP跟踪信息（默认情况下，最后100个HTTP请求 - 响应交换）。
* info(默认开启)     显示任意应用信息。
* integrationgraph   显示Spring Integration图。
* loggers       修改日志级别 实时
* liquibase     显示已应用的任何Liquibase数据库迁移。
* metrics      显示当前应用程序的“指标”信息。 线程 内存等
* mappings  显示所有`@RequestMapping`路径的整理列表。
* scheduledtasks  显示应用程序中的计划任务。
* sessions      允许从Spring Session支持的会话存储中检索和删除用户会话。使用Spring Session对响应式Web应用程序的支持时不可用。
* shutdown    允许应用程序正常关闭。
* threaddump   执行线程转储。

web应用附加节点:

* heapdump   返回hprof堆转储文件
* jolokia   通过http公开jmx bean
* logfile    返回日志文件的内容（如果已设置`logging.file`或`logging.path`属性）。支持使用HTTP `Range`标头检索部分日志文件的内容
* prometheus   以可以由Prometheus服务器抓取的格式公开指标

## 具体文档

https://docs.spring.io/spring-boot/docs/2.1.6.RELEASE/actuator-api//html/

# 配置

## 配置端点

端点自动缓存对不带任何参数的读取操作的响应。要配置端点缓存响应的时间量，请使用其`cache.time-to-live`属性。以下示例将`beans`端点缓存的生存时间设置为10秒：

**application.properties。** 

```
management.endpoint.beans.cache.time-to-live = 10s
```