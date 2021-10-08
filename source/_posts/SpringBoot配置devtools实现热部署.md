---
title: SpringBoot配置devtools实现热部署
date: 2018-10-05 15:45:18
tags: [SpringBoot]
---

>参考:   
>1.[springboot 热部署的两种方式](http://www.cnblogs.com/a8457013/p/8065489.html)  
>2.[Spring DevTools 介绍](https://blog.csdn.net/isea533/article/details/70495714)  
>3.[使用spring-boot-devtools进行热部署以及不生效的问题解决](https://blog.csdn.net/u012190514/article/details/79951258)

<!--more-->

#  devtools
# Pom.xml中直接添加依赖即可

```
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-devtools</artifactId>
      <scope>true</scope>
      <optional>true</optional>
      <!-- optional=true,依赖不会传递，该项目依赖devtools；之后依赖myboot项目的项目如果想要使用devtools，需要重新引入 -->  
    </dependency>
```

# 添加spring-boot-maven-plugin

```
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <configuration>
                <fork>true</fork>    
                 <!--fork :  如果没有该项配置，这个devtools不会起作用，即应用不会restart -->
            </configuration>
        </plugin>
    </plugins>
</build>
```

测试方法

修改类-->保存：应用会重启

修改配置文件-->保存：应用会重启

修改页面-->保存：应用会重启，页面会刷新（原理是将spring.thymeleaf.cache设为false）

----


# 如果使用如果使用`Intellij IEDA`

**<font color="red">Tips:</font>**需要到设置里将`project automatically`勾选上；`File->Setting->Build,…->Compiler`  将右侧`project automatically`勾上

`Intellij IEDA` 使用`ctrl+shift+a` 快捷键搜索`Registry`，选择搜索出来的第一个

Mac下快捷键是 `command+ shift+ a` 

找到`compiler.automake.allow.when.app.running`，勾上开启此功能即可



**两处设置不更改的话，Intellij IEDA可能无法生效**



