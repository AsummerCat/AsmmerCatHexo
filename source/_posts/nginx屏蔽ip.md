---
title: nginx屏蔽ip
date: 2019-04-19 07:29:40
tags: [nginx]
---

# nginx屏蔽ip

利用nginx屏蔽ip来实现防止采集

## 查找要屏蔽的ip

```
awk '{print $1}' nginx.access.log |sort |uniq -c|sort -n
```

nginx.access.log 为日志文件

会到如下结果，前面是ip的访问次数，后面是ip

```
13610 202.112.113.192
  95772 180.169.22.135
 337418 219.220.141.2
 558378 165.91.122.67
```

<!--more-->

## 新建屏蔽ip文件

在nginx的安装目录下面,新建屏蔽ip文件，命名为blockip.conf，以后新增加屏蔽ip只需编辑这个文件即可。 加入如下内容

```
deny 165.91.122.67; 
```

## 在nginx的配置文件nginx.conf中加入如下配置

在nginx的配置文件nginx.conf中加入如下配置，可以放到http, server, location,   limit_except语句块，需要注意相对路径，本例当中nginx.conf，blocksip.conf在同一个目录中。

```
include blockip.conf; 
```

## 重启一下nginx的服务

重启一下nginx的服务：`/usr/local/nginx/nginx -s reload` 就可以生效了。

# 高级用法

屏蔽ip的配置文件既可以屏蔽单个ip，也可以屏蔽ip段，或者只允许某个ip或者某个ip段访问。

## 屏蔽单个ip访问

```
deny IP; 
```

## 允许单个ip访问

```
allow IP; 
```

## 屏蔽所有ip访问

```
deny all;
```

## 允许所有ip访问

```
allow all;
```

## 屏蔽整个段即从123.0.0.1到123.255.255.254访问的命令

```
deny 123.0.0.0/8
```

## 屏蔽IP段即从123.45.0.1到123.45.255.254访问的命令

```
deny 124.45.0.0/16
```

## 屏蔽IP段即从123.45.6.1到123.45.6.254访问的命令

```
deny 123.45.6.0/24
```

如果你想实现这样的应用，除了几个IP外，其他全部拒绝，  
那需要你在blockip.conf中这样写

```
allow 1.1.1.1; 
allow 1.1.1.2;
deny all; 
```

单独网站屏蔽IP的方法，把include blocksip.conf; 放到网址对应的在server{}语句块，  
所有网站屏蔽IP的方法，把include blocksip.conf; 放到http {}语句块。

