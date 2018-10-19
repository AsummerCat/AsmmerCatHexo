---
title: linux环境下安装maven
date: 2018-10-18 16:22:49
tags: [linux,maven]
---

# 安装wget命令

 `yum -y install wget`
 
 <!--more-->
 
#  下载maven安装包
`wget http://mirrors.hust.edu.cn/apache/maven/maven-3/3.5.4/binaries/apache-maven-3.5.4-bin.tar.gz`

# 解压缩maven

`tar -zxvf apache-maven-3.5.4-bin.tar.gz`

# 配置maven环境变量


```
vi /etc/profile

添加环境变量

export MAVEN_HOME=/home/apache-maven-3.5.4
export MAVEN_HOME
export PATH=$PATH:$MAVEN_HOME/bin

编辑之后记得使用source /etc/profile命令是改动生效。
```

# 验证结果

在任意路径下执行mvn -version验证命令是否有效。

正常结果如下，能够看到当前maven及jdk版本。