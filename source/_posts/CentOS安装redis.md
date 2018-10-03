---
title: CentOS安装redis
date: 2018-09-29 22:41:22
tags: [linux,redis]
---
参考:
>https://www.linuxidc.com/Linux/2016-08/134003.htm
>https://www.cnblogs.com/limit1/p/9045183.html  
>http://www.runoob.com/redis/redis-install.html
# 1.官网
` http://redis.io/download`

---

<!--more-->

# 2.安装gcc编译
因为后面安装redis的时候需要编译，所以事先得先安装gcc编译，

`yum install gcc-c++`

---

# 4.下载，解压缩和编译Redis

```
$ wget http://download.redis.io/releases/redis-4.0.11.tar.gz
$ tar xzf redis-4.0.11.tar.gz
$ cd redis-4.0.11
$ make
$ cd src
$ make install PREFIX=/usr/local/redis
```
make这一步可能会报错，如果报错，可以尝试使用如下命令来编译：

make MALLOC=libc

　　编译好的二进制文件会放到src/目录下，可以看到有redis-server和redis-cli，这是redis的服务端可客户端，我们到时候可以直接运行这两个文件即可启动服务端和客户端，下面再说。另外还有一些其他配置文件。我们会觉得这有点乱，所以我们一般会自己新建一个目录专门存放命令和配置。

---

# 5.移动文件、便于管理

当然，你也可以不这么做~不过建议自己管理一下命令和配置

```
   移动配置文件到安装目录下  
　　cd ../  
　　mkdir /usr/local/redis/etc
　　mv redis.conf /usr/local/redis/etc
```

---

# 6.配置redis为后台启动
`vi /usr/local/redis/etc/redis.conf //将daemonize no 改成daemonize yes`


---

# 7.将redis加入到开机启动
```
vi /etc/rc.local 
//在里面添加内容：
/usr/local/redis/bin/redis-server /usr/local/redis/etc/redis.conf (意思就是开机调用这段开启redis的命令)
```

---

# 8.开启redis服务端
```
/usr/local/redis/bin/redis-server /usr/local/redis/etc/redis.conf 

	ps -ef | grep redis    //查看是否启动成功 
	netstat -tunpl | grep 6379   //查看该端口有没有占用 
```

---

# 9.启动redis客户端
redis客户端命令也在bin目录下，是redis-cli文件，运行一下即可启动redis客户端：

`./redis-cli`  

随便往里面插入一个name为eson15测试一下，可以正常获取，说明客户端没有问题。退出客户端的话直接quit即可。

---

# 10.关闭redis服务

关闭redis服务的话直接使用如下命令即可：

`pkill redis-server`


---

# 11.修改默认密码
我们可以通过以下命令查看是否设置了密码验证：

```
127.0.0.1:6379> CONFIG get requirepass
1) "requirepass"
2) ""
```
默认情况下 requirepass 参数是空的，这就意味着你无需通过密码验证就可以连接到 redis 服务。

```
127.0.0.1:6379> CONFIG set requirepass "jingbaobao"
OK
127.0.0.1:6379> CONFIG get requirepass
1) "requirepass"
2) "jingbaobao"
```
设置密码后，客户端连接 redis 服务就需要密码验证，否则无法执行命令。  
AUTH 命令基本语法格式如下：

`127.0.0.1:6379> AUTH password`

```
127.0.0.1:6379> AUTH "jingbaobao"
OK
127.0.0.1:6379> SET mykey "Test value"
OK
127.0.0.1:6379> GET mykey
"Test value"
```

---

# 12.常用命令
```
	ps -ef | grep redis    //查看是否启动成功 
	netstat -tunpl | grep 6379   //查看该端口有没有占用 

　　redis-server /usr/local/redis/etc/redis.conf //启动redis

　　pkill redis  //停止redis

　　卸载redis：

　　　　rm -rf /usr/local/redis //删除安装目录

　　　　rm -rf /usr/bin/redis-* //删除所有redis相关命令脚本

　　　　rm -rf /root/download/redis-4.0.4 //删除redis解压文件夹
```

