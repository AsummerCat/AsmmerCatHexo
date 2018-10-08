---
title: SpringBoot的构建
date: 2018-10-05 13:13:54
tags: SpringBoot
---

# 1.项目创建
>参考:  
>1.[Spring Boot 属性配置和使用](https://blog.csdn.net/isea533/article/details/50281151)  推荐 比较详细  
>2.[Spring Boot 集成MyBatis](https://blog.csdn.net/isea533/article/details/50359390)  

<!--more-->

初次接触，我们先来看看如何创建一个Spring Boot项目，这里以IntelliJ IDEA为例,
首先创建一个项目，创建时选择Spring Initializr，然后Next，如下图：

![](/img/SpringBoot/2018-10-5/SpringBoot1.png)

>然后填写一些基本信息:

![](/img/SpringBoot/2018-10-5/SpringBoot2.png)

>填写项目使用到的技术，右上角选择SpringBoot版本,下面勾选上Web就可以了，如下图：

![](/img/SpringBoot/2018-10-5/SpringBoot3.png)

最后填写项目存放的路径 这样一个项目就构建起来了

**<font color="red">需要注意的是启动类需要放在最外层,不然有可能会出现各种各样的问题</font>**

#### 1.1 常驻云服务器

运行命令：` nohup java -jar helloworld.jar &`

nohup的意思不挂服务，常驻的意思，除非云服务器重启，那就没法了；最后一个&表示执行命令后要生成日志文件nohup.out

---

# 2.Spring Boot的配置文件
Spring Boot使用一个全局的配置文件`application.properties`或者`application.yml`，配置文件放在`src/main/resources`目录下。`properties`是我们常见的一种配置文件，Spring Boot不仅支持`properties`这种类型的配置文件，也支持yaml语言的配置文件，我这里以properties类型的配置文件为例来看几个案例。

##### 2.1 修改Tomcat默认端口和默认访问路径
Tomcat默认端口是8080，我将之改为8081，默认访问路径是`http://localhost:8080`，我将之改为`http://localhost:8081/hello`,我们来看看这两个需求要怎么样通过简单的配置来实现。 
很简单，在`application.properties`文件中添加如下代码：

```
server.context-path=/hello   #根地址
server.port=8081    #端口号

```
---

#### 2.3编码配置
因为中文不做特殊处理会乱码，处理方式为继续在application.properties中添加如下代码

 ```
server.tomcat.uri-encoding=UTF-8
spring.http.encoding.charset=UTF-8
spring.http.encoding.enabled=true
spring.http.encoding.force=true
spring.messages.encoding=UTF-8
 ```

---
#### 2.3 常规属性配置
如果我们使用了Spring Boot，这项工作将会变得更加简单，我们只需要在application.properties中定义属性，然后在代码中直接使用@Value注入即可。 
如下：
```
book.age=18
book.name=小明
```


然后在变量中通过@Value直接注入就行了，如下：

```
 @Value(value = "${book.age}")
    private Integer age;
    @Value("${book.name}")
    private String name;
```

---


# 3.日志配置
SpringBoot中有一个自带的日志Logback;
当然如果有需要我们可以手动配置日志级别以及日志输出位置，相比于我们在Spring容器中写的日志输出代码，这里的配置简直就是小儿科了，只需要在`application.properties`中添加如下代码：

```
logging.file=/demo/springboot/log.log
logging.level.org.springframework.web=debug

```

---

# 4.Profile配置问题

在Spring Boot 中系统提供了更为简洁的方式。全局Profile配置我们使用application-{profile}.properties来定义，然后在application.properties中通过spring.profiles.active来指定使用哪个Profile。  
![](/img/SpringBoot/2018-10-5/SpringBoot4.png)

```
application-prod.properties:

`server.port=8081`

application-dev.properties:

`server.port=8080`

然后在`application.properties`中进行简单配置，如下：

`spring.profiles.active=dev`

这个表示使用开发环境下的配置。然后运行项目，我们得通过8080端口才可以访问

如果想换为生产环境，只需要把spring.profiles.active=dev改为spring.profiles.active=prod  

即可，当然访问端口这是也变为8081了

```

---