---
title: linux安装Tomcat8.5
date: 2018-09-30 22:12:12
tags: linux
---

# 下载Tomcat
>http://tomcat.apache.org/

<!--more-->

# wget命令下载到你的目录

>wget http://mirrors.shu.edu.cn/apache/tomcat/tomcat-8/v8.5.34/bin/apache-tomcat-8.5.34.tar.gz

# 解压
>ar -zxvf apache-tomcat-8.5.34.tar.gz


# 重命名
>mv apache-tomcat-8.5.34/ tomcat8.5.34  
>cp -r tomcat8.5.34/ /usr/local/tomcat


# 配置服务脚本
配置 Tomcat 服务

```

新建服务脚本
[root@localhost ~]# vim /etc/init.d/tomcat

添加脚本内容
#!/bin/bash
# description: Tomcat8 Start Stop Restart
# processname: tomcat8
# chkconfig: 234 20 80

CATALINA_HOME=/usr/local/tomcat/tomcat8.5.34

case $1 in
        start)
                sh $CATALINA_HOME/bin/startup.sh
                ;;
        stop)
                sh $CATALINA_HOME/bin/shutdown.sh
                ;;
        restart)
                sh $CATALINA_HOME/bin/shutdown.sh
                sh $CATALINA_HOME/bin/startup.sh
                ;;
        *)
                echo 'please use : tomcat {start | stop | restart}'
        ;;
esac
exit 0

最后给他执行权限:
chmod 755 tomcat

执行脚本，启动、停止 和 重启服务。
启动：service tomcat start
停止：service tomcat stop
重启：service tomcat restart
```

# 开机启动

```
Tomcat 配置开机自启动

向chkconfig添加 tomcat 服务的管理
[root@localhost ~]# chkconfig --add tomcat

设置tomcat服务自启动
[root@localhost ~]# chkconfig tomcat on

查看tomcat的启动状态
[root@localhost ~]# chkconfig --list | grep tomcat

状态如下：
[root@localhost ~]# chkconfig –list | grep tomcat

tomcat 0:off 1:off 2:on 3:on 4:on 5:on 6:off

关闭tomcat服务自启动：chkconfig tomcat off

删除tomcat服务在chkconfig上的管理：chkconfig –del tomcat
```

# 启动错误
## 第一个
>错误提示:Neither the JAVA_HOME nor the JRE_HOME environment variable is defined 完美解决（tomcat error）

原因：
因为启动tomcat会调用tomcat安装文件中的startup.bat，而它调用了catalina.bat则调用了setclasspath.bat。因此需要在setclasspath.bat的开头手动声明环境变量。

解决方案：
用vim打开tomcat的bin目录下的setclasspath.sh，添加JAVA_HOME和JRE_HOME两个环境变量，两个环境变量路径为您安装的java JDK的路径。
windows下将export改为set即可。

>export JAVA_HOME=/usr/local/java/jdk1.8.0_181
>
 export JRE_HOME=/usr/local/java/jdk1.8.0_181/jre

保存并且退出即可。
再次使用service tomcat start没报错，如下图所示：

成功用service tomcat start开启tomcat服务。

## 第二个
>错误提示:Tomcat启动时卡在“INFO: Deploying web application directory ......”的解决方法


第一次遇到Tomcat在Linux服务器启动卡住的情况，情况很简单，tomcat启动以后卡在INFO: Deploying web application directory ......  
这句话，具体会卡多久就没测试了。google、baidu都没找到解决方法。  
幸亏UCloud的技术支持人员给出了解决方案。  
找到jdk1.x.x_xx/jre/lib/security/java.security文件，在文件中找到securerandom.source这个设置项，将其改为：  
securerandom.source=file:/dev/./urandom   
这时候根据修改内容就可以查到因为此原因不仅可以造成tomcat卡住，也会造成weblogic启动缓慢，

linux或者部分unix系统提供随机数设备是/dev/random 和/dev/urandom ，两个有区别， 
urandom安全性没有random高，但random需要时间间隔生成随机数。  
jdk默认调用random。

---------------------
