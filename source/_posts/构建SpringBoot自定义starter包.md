---
title: 构建SpringBoot自定义starter包
date: 2020-04-16 15:10:07
tags: [SpringBoot]
---

# 构建SpringBoot自定义starter包

在springboot中，使用的最多的就是starter。starter可以理解为一个可拔插式的插件

自定义创建的方法也是很简单的

## demo地址

[demo地址](https://github.com/AsummerCat/Hello-Spring-Cloud-Starter)

比如:

## 先创建一个maven项目 

![](/img/2020-04-16/创建maven.png)

<!--more-->

## 注意一点

```
maven的打包格式要修改为jar
 <packaging>jar</packaging>
```

## 大致逻辑

1. 创建spring自动装配类
2. 创建配置文件 并且开启自动导入
3. 编写业务逻辑方法
4. check并且加入到Spring容器中
5. 编写spring.factories 导入我们自动装配类
6. 打包到仓库
7. 项目引用

## 导入自动装配的相关包

第一个依赖 主要是为了自动装配
第二个依赖 主要是为编译器配置的 可以根据properties 鼠标右键 点到用这个属性的类上个

```java
<dependencies>
      <dependency>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-autoconfigure</artifactId>
          <version>2.0.0.RELEASE</version>
      </dependency>
      <dependency>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-configuration-processor</artifactId>
          <version>2.0.0.RELEASE</version>
          <optional>true</optional>
      </dependency>
      </dependencies>
```

## 添加配置文件信息

```java
package com.linjingc.hello.config;

import org.springframework.boot.context.properties.ConfigurationProperties;

@ConfigurationProperties(prefix = "spring.hello")
public class HelloProperties {
	private String name = "小明";
	private int age = 20;
	private String address = "厦门市湖里区";

	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}

	public int getAge() {
		return age;
	}

	public void setAge(int age) {
		this.age = age;
	}

	public String getAddress() {
		return address;
	}

	public void setAddress(String address) {
		this.address = address;
	}
}

```



## 业务实现功能类

```java
public class HelloService {
	private HelloProperties helloProperties;

	public HelloService(HelloProperties helloProperties) {
		this.helloProperties = helloProperties;
	}

	public String heelloWord() {
		String say = helloProperties.getName() + "对世界说" + "我" + helloProperties.getAge() + "岁了" + ",来自" + helloProperties.getAddress();
		return say;
	}
}

```

## 写一个入口类 也是就当前自动装配的入口类

```java
@Configuration
//当类路径classpath下有指定的类的情况下进行自动配置
@ConditionalOnClass(HelloService.class)
//开启读取自动配置
@EnableConfigurationProperties(HelloProperties.class)
//如果获取不到属性配置 则使用默认的
@ConditionalOnProperty(prefix = "spring.hello", value = "enabled", matchIfMissing = true)
public class HelloAutoConfiguration {
	@Autowired
	private HelloProperties properties;


	@Bean
	// 当Spring Context中不存在该Bean时，自动配置HelloService类
	@ConditionalOnMissingBean(HelloService.class)
	public HelloService helloService() {
		HelloService helloService = new HelloService(properties);
		return helloService;
	}
	}
```

## 其他注解部分

```java
	/**
	 * @See @ConditionalOnClass：当类路径classpath下有指定的类的情况下进行自动配置
	 *
	 * @See @ConditionalOnMissingBean:当容器(Spring Context)中没有指定Bean的情况下进行自动配置
	 *
	 * @See @ConditionalOnProperty(prefix = “example.service”, value = “enabled”, matchIfMissing = true)，当配置文件中example.service.enabled=true时进行自动配置，如果没有设置此值就默认使用matchIfMissing对应的值
	 *
	 * @See @ConditionalOnMissingBean，当Spring Context中不存在该Bean时。
	 *
	 * @See @ConditionalOnBean:当容器(Spring Context)中有指定的Bean的条件下
	 *
	 * @See @ConditionalOnMissingClass:当类路径下没有指定的类的条件下
	 *
	 *  @See @ConditionalOnExpression:基于SpEL表达式作为判断条件
	 *
	 * @See @ConditionalOnJava:基于JVM版本作为判断条件
	 *
	 * @See @ConditionalOnJndi:在JNDI存在的条件下查找指定的位置
	 *
	 * @See @ConditionalOnNotWebApplication:当前项目不是Web项目的条件下
	 *
	 * @See @ConditionalOnWebApplication:当前项目是Web项目的条件下
	 *
	 * @See @ConditionalOnResource:类路径下是否有指定的资源
	 *
	 * @See @ConditionalOnSingleCandidate:当指定的Bean在容器中只有一个，或者在有多个Bean的情况下，用来指定首选的Bean
	 */

```

## 创建spring.factories

这边的话是让Spring能找到项目外的自动装配的内容

### 路径

```
路径: resoures/META-INF/spring.factories 需要手动创建
```

### 内容

这边的话 需要把我们自定义的自动装配类 指向EnableAutoConfiguration;

```
org.springframework.boot.autoconfigure.EnableAutoConfiguration=com.linjingc.hello.HelloAutoConfiguration
```

## 后面打包 和安装maven就可以了

```
mvn --insert
```

## 项目引用

```java
<dependency>
            <groupId>com.linjingc.hello</groupId>
            <artifactId>hello-spring-cloud-starter</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>

```

## 修改配置文件

因为我们的配置参数 配置了从spring.hello下获取

如果不填写的话就是使用默认的配置 ,如需修改如下

```
spring:
  hello:
    address: 北京市
    name: 小东
    age: 100
```

