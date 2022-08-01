---
title: Gradle构建springboot多模块项目
date: 2019-09-27 11:55:29
tags: [Gradle]
---

# Gradle构建springboot多模块项目

[demo地址](https://github.com/AsummerCat/gradle-demo)

# 父项目的构建

前期步骤跟pom差不多 构建一个

gradle的项目 项目内置保留一个settings.gradle的配置文件 这个就是父项目了

![父项目包结构](/img/2019-09-27/2.png)

![父项目包结构](/img/2019-09-27/1.png)

<!--more-->

有两个配置文件

## settings.gradle

这个是用来维护父子项目关系的

```
rootProject.name = 'gradle-demo'
include 'entiry'
include 'web'
```

## build.gradle

维护包和子模块包类似pom.xml

```
plugins {
    id 'org.springframework.boot' version '2.1.8.RELEASE'           //spring提供的spring boot插件,主要用到了其依赖管理的功能.
    id 'io.spring.dependency-management' version '1.0.8.RELEASE'
}




allprojects {
    group 'com.linjingc'
    version '1.0-SNAPSHOT'
    apply plugin: 'java'
    apply plugin: 'idea'
    apply plugin: 'org.springframework.boot'                        //spring boot插件
    apply plugin: 'io.spring.dependency-management'                 //实现maven的依赖统一管理功能

    sourceCompatibility = '1.8'
    targetCompatibility  = '1.8'

    tasks.withType(JavaCompile) { options.encoding = "UTF-8" }

    // java编译的时候缺省状态下会因为中文字符而失败
    [compileJava,compileTestJava,javadoc]*.options*.encoding = 'UTF-8'

    configurations {
        compileOnly {
            extendsFrom annotationProcessor
        }
    }

   //仓库配置
    repositories {
        mavenCentral()
    }
    
}

// 所有子项目的通用配置
subprojects {
    dependencies {
        testCompile group: 'junit', name: 'junit', version: '4.12'
        compileOnly 'org.projectlombok:lombok:1.18.8'
        annotationProcessor 'org.projectlombok:lombok:1.18.8'
    }
}
```

# 子项目的构建

![子项目包结构](/img/2019-09-27/3.png)



这边需要注意的是只有一个配置文件`build.gradle` 其他多余的内容全部手动删除  如:gradle.bat之类的

## web模块的配置

这里因为是一个web模块 需要引入单独的包 就直接操作 其他的都会继承父类拿包

```
plugins {
    id 'war'
}

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
    compile project(':entiry')
}

```

# entiry模块

这边的配置文件 可以不填写 因为没东西

放空就好了



这样就基本搭建完成了

# 关联父子项目

这边主要靠 父项目的settings.gradle

