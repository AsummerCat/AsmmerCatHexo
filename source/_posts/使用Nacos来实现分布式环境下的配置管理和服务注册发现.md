---
title: 使用Nacos来实现分布式环境下的配置管理和服务注册发现
date: 2019-01-19 10:12:48
tags: [Nacos,SpringCloudAlibaba]
---

[服务发现文档](https://github.com/spring-cloud-incubator/spring-cloud-alibaba/wiki/Nacos-discovery)

[配置信息文档 替代SpringCloudConfig](https://github.com/spring-cloud-incubator/spring-cloud-alibaba/wiki/Nacos-config)

Spring Cloud Alibaba项目由两部分组成：阿里巴巴开源组件和阿里云产品组件，旨在为Java开发人员在使用阿里巴巴产品的同时，通过利用 Spring 框架的设计模式和抽象能力，注入Spring Boot和Spring Cloud的优势。

> Spring Cloud Alibaba 项目是由阿里巴巴维护的社区项目。
>
> **注意**： 版本 0.2.0.RELEASE 对应的是 Spring Boot 2.x 版本，版本 0.1.0.RELEASE 对应的是 Spring Boot 1.x 版本.

<!--more-->

其中阿里巴巴开源组件的命名前缀为`spring-cloud-alibaba`，提供了如下特性：

### **| 服务发现**

实现了 Spring Cloud common 中定义的 registry 相关规范接口，引入依赖并添加一些简单的配置即可将你的服务注册到Nacos Server中，并且支持与Ribbon的集成。

### **| 配置管理**

实现了 `PropertySoureLocator` 接口，引入依赖并添加一些简单的配置即可从 Nacos Server 中获取应用配置并设置在 Spring 的 Environment 中，而且无需依赖其他组件即可支持配置的实时推送和推送状态查询。

### **| 高可用防护**

默认集成了 Servlet、RestTemplate、Dubbo、RocketMQ 的限流(Flow Control)降级(Circuit Breaking and Concurrency)，只需要引入依赖即可完成限流降级的集成动作，并支持在应用运行状态下通过 Sentinel 控制台来实时修改限流降级的策略和阈值。



# 正文

