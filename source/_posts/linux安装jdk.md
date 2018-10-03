---
title: linux安装jdk
date: 2018-09-26 22:34:21
tags: linux
---
# 1.yum安装
**centos7安装Openjdk1.8**

#### 1.检查是否存在旧版本

```
安装之前先检查一下系统有没有自带open-jdk

命令：

rpm -qa |grep java

rpm -qa |grep jdk

rpm -qa |grep gcj

```
<!--more-->
```
如果没有输入信息表示没有安装。

如果安装可以使用
rpm -qa | grep java | xargs rpm -e --nodeps 
批量卸载所有带有Java的文件  这句命令的关键字是java
```
---

#### 2.检索包含java的列表
```
首先检索包含java的列表

yum list java*
 
检索1.8的列表

yum list java-1.8*   

安装1.8.0的所有文件

yum install java-1.8.0-openjdk* -y

```
---
#### 3.安装完成

```
使用命令检查是否安装成功

java -version



到此安装结束了。这样安装有一个好处就是不需要对path进行设置，
自动就设置好了


```




# 2.普通安装 

```
第一种：安装tar.gz类型的jdk
下载jdk-7u76-linux-x64.tar.gz 


上传到linux机器上（/usr/local）
tar -xvf  jdk-7u76-linux-x64.tar.gz 
会解压出 jdk1.7.0_79的文件夹
配置路径 vi /etc/profile
export JAVA_HOME=/usr/local/jdk1.7.0_79
export JRE_HOME=$JAVA_HOME/jre
export PATH=$JAVA_HOME/bin:$PATH
export CLASSPATH=./:JAVA_HOME/lib:$JRE_HOME/lib
重启系统：shutdown -r now
测试：java -version
显示出版本信息


``` 