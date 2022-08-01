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

```
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

在tomcat\conf\server.xml中的`<host></host>`

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

```
<?xml version="1.0" encoding="UTF-8"?> 
<Context docBase="D:\demo1\web" reloadable="true" /> 
```

# 关闭shutdown端口

shown端口是写在Server参数里的，直接去掉是不管用，也是会默认启动的，一般在安全设置时候建议把端口修改为其他端口，shutdown修改为其他复杂字串。实际上这个端口是可以直接屏蔽不监听的。

这边可以通过 

```
telnet 127.0.0.1 8080 
---> shotdown 
进行关闭的
```

修改server.xml中的

```
<Server port="8005" shutdown="SHUTDOWN">
```

## 方式一 更改port的参数值

进入Tomcat文件的conf文件夹的server.xml中，将port的参数值更改为-1：

```
<Server port="-1" shutdown="SHUTDOWN">
```

## 方式二 更改shutdown的参数值

进入Tomcat文件的conf文件夹的server.xml中，将shutdown的参数值更改为QWEASD：

```
<Server port="8005" shutdown="QWEASD">
```



# 隐藏版本信息

黑客可以根据指定版本漏洞进行攻击

```
进入tomcat根目录->lib->catalina.jar文件
```

```
进入org/apache/catalina/util
编辑文件ServerInfo.properties
```

```
修改以下内容:

server.info=Apache Tomcat/8.5.28   ->改为 99999
server.number=8.5.28.0             -> 10086
server.built=Feb 6 2018 23:10:25 UTC   -> Feb 6 2010 23:10:25 UTC 
```

# 禁用Tomcat管理界面

## 重命名tomcat目录下的ROOT 

进入Tomcat文件夹下的webapps文件夹，将其中的ROOT文件重命名，然后新建一个ROOT文件夹即可

# 启动cookie的HttpOnly属性



# 优化配置

## 缓存优化 (niginx,gzip)

