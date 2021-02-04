---
title: apollo整合springboot(三)
date: 2021-02-04 10:25:17
tags: [apollo,分布式配置中心]
---

[demo地址](https://github.com/AsummerCat/apollodemo)

# 客户端使用

注意服务器是使用eureka的
## 导入pom.xml
```
        <dependency>
            <groupId>com.ctrip.framework.apollo</groupId>
            <artifactId>apollo-client</artifactId>
            <version>1.7.0</version>
        </dependency>
        
      <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-context</artifactId>
        </dependency>
        
```
<!--more-->
## 创建app.properties
在`resources下创建一个META-INF文件夹再创建app.properties`
内容:
```
#自定义的appid名称，区分不同的应用
app.id=SampleApp
```
## 修改application.yml
添加apollo的相关配置
```
apollo:
  #eureka配置中心地址
  meta: http://localhost:8080
  cacheDir: /opt/data/test
  bootstrap:
    enabled: true
    eagerLoad:
      enabled: true

```

## 代码中开启apollo注解

## 使用 
就跟spring cloud config一样的使用
```
@value(#{xxx属性:默认值})

例如:@Value("${timeout:100}")
```