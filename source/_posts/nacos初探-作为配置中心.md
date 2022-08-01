---
title: nacos初探--作为配置中心
date: 2019-01-21 10:57:13
tags: [Nacos,SpringCloudAlibaba]
---

Nacos 支持基于 DNS 和基于 RPC 的服务发现（可以作为SpringCloud的注册中心）、动态配置服务（可以做配置中心）、动态 DNS 服务。

官方网址：[http://nacos.io](http://nacos.io/)

官方文档:[Nacos config](https://github.com/spring-cloud-incubator/spring-cloud-alibaba/wiki/Nacos-config)

# nacos作为注册中心

先在官网上下载nacos中间件 下面教程有启动步骤

```
https://nacos.io/zh-cn/docs/quick-start.html
```

程序启动默认占用的端口是8848（珠穆朗玛峰的高度），我们可以对端口进行修改，用编辑器打开bin目录下的startup.cmd文件 添加一行代码

```
set "JAVA_OPT=%JAVA_OPT% --server.port=9090
```

修改启动端口 为9090 

还可以在conf文件下的application.properties中添加

```
server.port=9090
```



# 开始正文 

先新建一个springboot项目

## 添加pom

```
<!-- dependency management 0.2.0 springboot	2.0  0.1 1.0+-->
	<dependencyManagement>
		<dependencies>
			<dependency>
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-dependencies</artifactId>
				<version>Finchley.SR1</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>
			<dependency>
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-alibaba-dependencies</artifactId>
				<version>0.2.0.RELEASE</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>
		</dependencies>
	</dependencyManagement>

<!-- dependencies -->
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
    </dependency>
    
    <!--这里是config-->
    <dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-config-server</artifactId>
    <version>0.2.1.RELEASE</version>
</dependency>
</dependencies>
```

## 创建bootstrap.yml

这边就不描述了 bootstrap.properties 启动在application之前

把nacos服务器添加进去

这边基本跟spring cloud config 差不多

```
spring.application.name=nacos-config
spring.cloud.nacos.config.server-addr=112.74.43.136:8848
    
```

这边基本上就算配置完成了 

**但是呢 需要注意的是 首先要在nacos中配置一下需要获取的值 不然启动报错** 

## 进入nacos中添加键