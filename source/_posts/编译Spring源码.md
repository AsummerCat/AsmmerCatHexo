---
title: 编译Spring源码
date: 2020-04-08 21:44:11
tags: [Spring,源码解析]
---

# 编译Spring源码

## 构建环境

```
1.下载Gradle二进制包
https://gradle.org/install/

2.配置gradle环境变量
变量名：GRADLE_HOME  地址：C:\gradle-6.2.2

追加Path
在系统变量 path中加入：%GRADLE_HOME%\bin;

```

## 编译

<!--more-->

```
进入 spring-framework 文件夹下，打开cmd，输入 .\gradlew :spring-oxm:compileTestJava 进行编译。
```

## 构建项目

```
1. idea勾选 
  Use auto-import （自动导入依赖）
  Create separate module per source set  （为每个资源集创建单独的模块）
  Use local gradle distribution （使用本地gradle）
  
  
```

## 建立测试module


```
在Project Structure中需要将spring-aspectj这个module除去，因为build报错。我在build的时候还有context下的money啥的也报错，同样exclude掉了。

```
使用gradle建立module，建立后
1. 在 mytest 的gradle配置文件中添加 compile(project(":spring-beans")) 依赖，便于下面进行容器 BeanFactory 的使用。
2. dependencies添加compile(project(":spring-beans"))

### 添加日志输出

Spring5使用了log4j2，想要输出日志信息，我们需要在自己定义的模块下配置配置文件。在 src/main/resources 文件夹下新建 log4j2.xml 文件，在 src/test/resources 目录下创建 log4j2-test.xml 文件。文件配置内容为：

```
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="DEBUG">
	<Appenders>
		<Console name="Console" target="SYSTEM_OUT">
			<PatternLayout pattern="%d{HH:mm:ss.SSS} [%t] %-5level %logger{36} - %msg%n" />
		</Console>
	</Appenders>
	<Loggers>
		<Logger name="org.springframework" level="DEBUG" />
		<Root level="DEBUG">
			<AppenderRef ref="Console" />
		</Root>
	</Loggers>
</Configuration>

```

## 这样就可以开始构建你的demo了 


```
AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(AppConfig.class);
		Person person = (Person) applicationContext.getBean("person");
		System.out.println(person.toString());
```