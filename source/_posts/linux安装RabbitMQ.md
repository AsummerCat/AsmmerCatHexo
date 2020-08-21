---
title: linux安装RabbitMQ
date: 2019-01-15 17:08:39
tags: [linux,RabbitMQ]
---

# RabbitMQ安装需要Erlang环境

由于RabbitMQ依赖Erlang， 所以需要先安装Erlang。

<!--more-->

## Erlang的安装方式大概有两种

### 从Erlang Solution安装(**推荐**)

```java
 # 添加erlang solutions源
 $ wget https://packages.erlang-solutions.com/erlang-solutions-1.0-1.noarch.rpm
 $ sudo rpm -Uvh erlang-solutions-1.0-1.noarch.rpm
 
 $ sudo yum install erlang

```

### 从EPEL源安装(这种方式安装的Erlang版本可能不是最新的，有时候不能满足RabbitMQ需要的最低版本)

```java
 # 启动EPEL源
 $ sudo yum install epel-release 
 # 安装erlang
 $ sudo yum install erlang  

```



# 安装RabbitMQ

## 先下载rpm

```java
wget http://www.rabbitmq.com/releases/rabbitmq-server/v3.6.6/rabbitmq-server-3.6.6-1.el7.noarch.rpm
```

## 安装

```java
yum install rabbitmq-server-3.6.6-1.el7.noarch.rpm 
```



# Tip

安装时如果遇到下面的依赖错误

```java
Error: Package: socat-1.7.2.3-1.el6.x86_64 (epel)
       Requires: libreadline.so.5()(64bit)
```

可以尝试先执行

```
sudo yum install socat
```



# 开启web管理接口

如果只从命令行操作RabbitMQ，多少有点不方便。幸好RabbitMQ自带了web管理界面，只需要启动插件便可以使用。

```java
$ sudo rabbitmq-plugins enable rabbitmq_management	
```

然后通过浏览器访问

[http://localhost:15672](https://link.jianshu.com/?t=http://localhost:15672)

输入用户名和密码访问web管理界面了。



# 配置RabbitMQ
关于RabbitMQ的配置，可以下载RabbitMQ的配置文件模板到/etc/rabbitmq/rabbitmq.config, 然后按照需求更改即可。
关于每个配置项的具体作用，可以参考官方文档。
更新配置后，别忘了重启服务哦！

## 开启用户远程访问
默认情况下，RabbitMQ的默认的guest用户只允许本机访问， 如果想让guest用户能够远程访问的话，只需要将配置文件中的loopback_users列表置为空即可，如下：

```
{loopback_users, []}
```


另外关于新添加的用户，直接就可以从远程访问的，如果想让新添加的用户只能本地访问，可以将用户名添加到上面的列表, 如只允许admin用户本机访问。

```
{loopback_users, ["admin"]}
```



创建一个新用户

```
sudo rabbitmqctl add_user username password
```

赋予角色

```
$ sudo rabbitmqctl set_user_tags username administrator
```

开启权限

```
sudo rabbitmqctl set_permissions -p / username ".*" ".*" ".*"
```

更新配置后，别忘了重启服务哦！



```
 sudo /sbin/service rabbitmq-server status  # 查看服务状态
```



![](/img/2019-1-15/rabbitMq1.png)

这里可以看到log文件的位置，转到文件位置，打开文件：

![](/img/2019-1-15/rabbitMq2.png)

这里显示的是没有找到配置文件，我们可以自己创建这个文件

```
cd /etc/rabbitmq/
vi rabbitmq.config
编辑内容如下：

[{rabbit, [{loopback_users, []}]}].
```

这里的意思是开放使用，rabbitmq默认创建的用户guest，密码也是guest，这个用户默认只能是本机访问，localhost或者127.0.0.1，从外部访问需要添加上面的配置。

```
二、可以通过 find / -name rabbitmq-defaults 查找rabbitmq-defaults文件，查看config文件的存储路径

三、通过 https://github.com/rabbitmq/rabbitmq-server/blob/master/docs/rabbitmq.conf.example 下载保存为rabbitmq.conf,将其放到config文件的存储路径中，例如/etc/rabbitmq/下
```





保存配置后重启服务：

```
rm rabbit\@mythsky.log 
service rabbitmq-server stop
service rabbitmq-server start
```

注意:记得要开放5672和15672端口



