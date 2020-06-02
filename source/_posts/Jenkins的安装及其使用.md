---
title: Jenkins的安装及其使用
date: 2019-01-23 11:27:14
tags: [Jenkins]
---

# Jenkins持续集成部署

Jenkins 是一个开源软件项目，旨在提供一个开放易用的软件平台，使软件的持续集成变得可能。

<!--more-->

# 安装

1.Git版本控制服务器

2.Tomcat发布服务器

3.Jenkins服务器(提前安装好Maven,Git,Jdk)

### 下载Jenkins War包，Jenkins官网 。

https://jenkins.io/download/ 根据自己需求下载 我这边直接下载war

然后放入tomcat中启动 这样就安装成功了

如何验证jenkins是否正确安装呢？只需要在浏览器中访问<http://localhost:8080/jenkins/> ，就能访问jenkins了。当然，ip和端口号得是你tomcat的IP和端口号。 

# 初始化配置

首次访问 会叫你配置密码

![](/img/2019-1-23/jenkins.png)

赋值粘贴 后 vi 就可以看到初始密码了 然后输入 就可以了

### 安装插件 

两种方式 第一种 推荐安装 第二种自定义安装

**安装Jenkins插件**

1.Email Extension Plugin (邮件通知)

2.GIT plugin (可能已经默认安装了)

3.Publish Over SSH (远程Shell)

** 安装方法：**

首页->系统管理->管理插件->可选插件->过滤(搜索插件名)->勾选->点击最下面直接安装即可(需要等待一段时间,详情可以看catalina.out日志变化)



其他内容参考: https://www.cnblogs.com/helenMemery/p/6646978.html

这些简单的部署就是这样的