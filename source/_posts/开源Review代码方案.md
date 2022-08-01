---
title: 开源Review代码方案
date: 2020-10-19 11:19:30
tags: [java]
---

# 利用**Sonarqube**实现 

Sonarqube 是一个用于代码质量管理的开放平台。通过插件机制，Sonarqube 可以集成不同的测试工具，代码分析工具，以及持续集成工具。

在对其他工具的支持方面，Sonarqube 不仅提供了对 IDE 的支持，可以在 Eclipse 和 IntelliJ IDEA 这些工具里联机查看结果；同时 Sonarqube 还对大量的持续集成工具提供了接口支持，可以很方便地在持续集成中使用 Sonar。

简单来说，Sonarqube就是一个代码质量检测工具，可以通过搭建服务端和使用客户端来对代码进行检测

<!--more-->

# **搭建Sonarqube服务器**

利用docker安装

```
docker search sonarqube

docker pull sonarqube

docker run -d --name sonarqube -p 9000:9000 -p 9092:9092 sonarqube

访问地址：http://localhost:9000
点击左侧的Log in登录，默认的登录用户名密码都是admin
```

# 客户端使用

## 方案一 纯idea插件查看

```
安装插件SonarLint并重启IDE
```

重启之后就要配置客户端连接的服务器了，服务器地址和账号密码填写我们之前本地搭建的信息

![](/img/2020-10-19/1.jpg)

点击Next的时候会需要创建Token

![](/img/2020-10-19/2.jpg)

点击`Create Token`跳转到我们生成Token的网页，这里我们输入admin创建Token

![](/img/2020-10-19/3.jpg)

复制生成的Token到idea里面，填写好即可

**验证和使用**

接下来就是验证使用了。我们在项目代码目录上右键

![](/img/2020-10-19/4.jpg)

会有SonarLint这个选项，点击第一个

![](/img/2020-10-19/5.jpg)

可以看到代码检测愉快的跑起来了。

![](/img/2020-10-19/6.jpg)

扫描结束以后，可以看到，很多不规范的代码都被扫出来了。

随便点开一个，比如说这个空方法

![](/img/2020-10-19/7.jpg)

并且右侧给出了对应的修复示例参考

![](/img/2020-10-19/8.jpg)

## 方案二 直接控制台+maven查看

登录管理台，点击Markerplace模块，安装中文包

![](/img/2020-10-19/9.jpg)

安装完成重启服务

![](/img/2020-10-19/10.jpg)

然后在项目里面加入以下maven依赖

```
<plugin>
  <groupId>org.sonarsource.scanner.maven</groupId>
  <artifactId>sonar-maven-plugin</artifactId>
  <version>3.7.0.1746</version>
</plugin>
```

展开项目的Maven选项,双击运行，执行完毕后就可以登录管理台查看了。

![](/img/2020-10-19/11.jpg)

打开管理台，你会发现产生了一个和你项目名一样的项目，并且各种代码质量指标都标注的清清楚楚！！

![](/img/2020-10-19/12.jpg)

我们点进去，点开bug选项随便一处，查看Bug

![](/img/2020-10-19/13.jpg)



果然扫出来了一处可能出现bug的代码，假设Get这个枚举对象为null的话，下面的对象getId()方法肯定会出现空指针了。