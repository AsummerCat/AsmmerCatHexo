---
title: tomcat调优方案
date: 2019-10-16 22:13:40
tags: [tomcat,调优]
---

# 配置内存

## 方式一 catalina.bat

在tomcat主目录 bin文件夹 下修改catalina.bat

在第二行添加如下代码：

```
set JAVA_OPTS=%JAVA_OPTS% -server -Xms512m -Xmx512m -XX:MaxNewSize=256m -XX:PermSize=512M -XX:MaxPermSize=512m
```



## 方式二 *start*up.bat

catalina.bat和startup.bat修改都行，因为startup.bat会调用catalina.bat

```java
解释一下各个参数：

-Xms1024M：初始化堆内存大小（注意，不加M的话单位是KB）

-Xmx1024M：最大堆内存大小

-XX:PermSize=256M：JVM初始分配的非堆内存, 不会被回收, 生产环境建议与maxPermSize相同

-XX:MaxPermSize=256M：JVM最大允许分配的非堆内存, 生产环境建议设置为256m以上

-XX:MaxNewSize=256M：JVM堆区域新生代内存的最大可分配大小(PermSize不属于堆区)

还有一个-server参数，是指启动jvm时以服务器方式启动，比客户端启动慢，但性能较好，大家可以自己选择。
```

<!--more--> 



# 热部署

替换class文件重新加载项目时就不用重启tomcat

## 方式一

在tomcat\conf\server.xml中的<host></host>

```
<Context debug="0" docBase="D:\demo1\web" path="/demo1"  reloadable="true"/>
```

或者

```
   <Context path="/myapp" docBase="myapp" debug="99" reloadable="true" crossContext="true"/>  
```

docBase:项目路径，可以使用绝对路径或相对路径，相对路径是相对于webapps 
path:访问项目的路径，如：http://127.0.0.1:8080/demo1 
reloadable:是否自动加载新增或改变的class文件. 
debug属性与这个Engine关联的Logger记录的调试信息的详细程度。数字越大，输出越详细。如果没有指定，缺省为0。 也就是程序异常时写入日志文件里的详细程度

crossContext 热加载

## 方式二

conf\Catalina\localhost中添加一个XML文件

这个xml的名称就是路径

```java
<?xml version="1.0" encoding="UTF-8"?> 
<Context docBase="D:\demo1\web" reloadable="true" /> 
```



# 2333

