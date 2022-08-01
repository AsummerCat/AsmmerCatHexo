---
title: Amazon Corretto的安装
date: 2019-01-03 00:18:59
tags: [openJdk]
---

# Amazon Corretto

Amazon Corretto 官网：https://aws.amazon.com/cn/corretto/

目前 Amazon 发布了 Corretto 的 Corretto 8 Preview 预览版本，它基于 OpenJDK 8 源码。

下载地址：https://docs.aws.amazon.com/zh_cn/corretto/latest/corretto-8-ug/downloads-list.html

<!--more-->

# 安装

**在 Amazon Linux 2 环境中安装 Amazon Corretto 8**

\1. 启用 Corretto 8 的 YUM 仓库

```
$ amazon-linux-extras enable corretto8
```

\2. 可以将 Amazon Corretto 8 安装为运行时环境（JRE）或完整开发环境（JDK），后者包含了运行时环境。

将 Amazon Corretto 8 安装为 JRE：

```
$ sudo yum install java-1.8.0-amazon-corretto
```

将 Amazon Corretto 8 安装为 JDK：

```
$ sudo yum install java-1.8.0-amazon-corretto-devel
```

> 安装位置是 /usr/lib/jvm/java-1.8.0-amazon-corretto.x86_64。

\3. 验证安装

在终端中，运行以下命令：

```
$ java -version
openjdk version "1.8.0_192"
OpenJDK Runtime Environment (build 1.8.0_192-amazon-corretto-preview-b12)
OpenJDK 64-Bit Server VM (build 25.192-b12, mixed mode)
```

\4. 卸载 Amazon Corretto 8

可以使用以下命令卸载 Amazon Corretto 8：

卸载 JRE：

```
$ sudo yum remove java-1.8.0-amazon-corretto
```

卸载 JDK：

```
$ sudo yum remove java-1.8.0-amazon-corretto-devel
```

# **Amazon Corretto 8 的 Docker 镜像**

\1. 建立 Amazon Corretto 8 的 Docker 镜像

```
$ docker build -t amazon-corretto-8 github.com/corretto/corretto-8-docker
```

命令完成后，将拥有一个名为 amazon-corretto-8 的镜像。

要在本地运行此镜像，请运行以下命令：

```
$ docker run -it amazon-corretto-8
```

还可以将此镜像推送到 Amazon ECR。

\2. 创建一个新的 Docker 镜像

可以使用 Amazon Corretto 8 Docker 镜像作为父镜像来创建新的 Docker 镜像：

创建Dockerfile，内容如下：

```
FROM amazon-corretto-8
RUN echo $' \
public class Hello { \
public static void main(String[] args) { \
System.out.println("Welcome to Amazon Corretto!"); \
} \
}' > Hello.java
RUN javac Hello.java
CMD ["java", "Hello"] 
```

构建新镜像：

```
$ docker build -t hello-app .
```

运行新镜像：

```
$ docker run hello-app
```

将获得以下输出。

Welcome to Amazon Corretto!