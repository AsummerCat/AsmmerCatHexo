---
title: SpringBoot打成一个war包
date: 2018-10-09 15:48:49
tags: [SpringBoot,tomcat]
---

# 1.修改pom.xml

#### 1.1 移除嵌入式tomcat插件

```
 <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <!-- 移除嵌入式tomcat插件 -->
    <exclusions>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
        </exclusion>
    </exclusions>
</dependency>


```

<!--more--> 


#### 1.2  El表达式

```
<dependency>
   <groupId>javax.servlet</groupId>
   <artifactId>jstl</artifactId>
   <version>1.2</version>
</dependency>
```

#### 1.3 添加servlet-api的依赖
二选一

```
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>javax.servlet-api</artifactId>
    <version>3.1.0</version>
    <scope>provided</scope>
</dependency>

```

```

<dependency>
    <groupId>org.apache.tomcat</groupId>
    <artifactId>tomcat-servlet-api</artifactId>
    <version>8.0.36</version>
    <scope>provided</scope>
</dependency>
```

#### 1.4 修改启动类，并重写初始化方法

在原本启动类上 继承 `SpringBootServletInitializer `

```
@Override
	protected SpringApplicationBuilder configure(SpringApplicationBuilder builder) {
		// 注意这里要指向原先用main方法执行的Application启动类
		return builder.sources(SpringbootandredisApplication.class);
	}

```

#### 1.5 打包部署

搞定~~~

