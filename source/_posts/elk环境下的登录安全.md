---
title: elk环境下的登录安全
date: 2019-01-20 19:54:07
tags: ELK日志分析框架
---

# 登录安全

我们实现的elk 并没有实现权限安全控制 所有人都可以访问

并不安全 ,所以我们要是利用nginx 实现权限安全的控制

因为 filebeat 或logstash是运行在应用端上的 所以这一部分并不需要有权限安全的控制

<!--more-->

# elasticsearch的登录

这边是需要利用`nginx` 来控制登录 

注意: elasticsearch设置登录密码后 logstash那边的配置文件中相应的也要设置密码

### 开启nginx 利用转发控制

#### 修改nginx配置文件

```java
server {
# 监听80端口
  listen       80;
# 基于域名的虚拟主机 所有来自linjingc.top的请求都会转发到该主机上
  server_name linjingc.top;
  location / {
# 虚拟主机认证命名
       auth_basic`"secret";
# 虚拟主机用户名密码认证数据库  (接下来配置)    
     auth_basic_user_file /data/nginx/db/passwd.db;
     proxy_pass http://localhost:9200;
     proxy_set_header Host $host:9200;
     proxy_set_header X-Real-IP $remote_addr;
     proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
     proxy_set_header Via "nginx";
  }
  # 关闭日志
  access_log off;
}
```

上面的配置表示将linjingc.top的请求，转发到服务器的9200端口，同时使用最基本的用户名、密码来认证。

#### 配置登录用户名，密码

```
htpasswd -c /data/nginx/db/passwd.db cat
```

注意passwd.db的路径要跟nginx配置中的一致，最后的cat为用户名，可以随便改，输入完该命令后，系统会提示输入密码，搞定后passwd.db中就有加密后的密码了，有兴趣的可以cat看下。

**提示：htpasswd是apache自带的小工具，如果找不到该命令，尝试用yum install httpd安装**

#### 关掉kibana端口的外网访问

用nginx转发后，一定要记得配置iptables之类的防火墙，禁止外部直接访问9200端口，这样就只能通过nginx来访问了

## 最后需要注意的elasticsearch设置了密码时候

logstash的配置文件也要进行相应修改

```
output {
        if [type] == "elk-log" {
                elasticsearch {
                        hosts => ["192.168.1.101:9200"]
                        index => "elk-log-%{+YYYY.MM.dd}"
                        //加入 user 和password
                        user => cat
                        password => password
                }
        }
}
```



# kibana的登录



这边也是需要利用`nginx` 来控制登录 

注意: elasticsearch设置登录密码后 kibana那边的配置文件中相应的也要设置密码

kibana是提供给外网访问的

**tips：kibana没有重启命令，要重启，只能ps -ef|grep node 查找nodejs进程，干掉重来。**

### 开启nginx 利用转发控制

#### 修改nginx配置文件

```java
server {
# 监听80端口
  listen       80;
# 基于域名的虚拟主机 所有来自linjingc.top的请求都会转发到该主机上
  server_name linjingc.top;
  location / {
# 虚拟主机认证命名
       auth_basic`"secret";
# 虚拟主机用户名密码认证数据库  (接下来配置)    
     auth_basic_user_file /data/nginx/db/passwd.db;
     proxy_pass http://localhost:5601;
     proxy_set_header Host $host:5601;
     proxy_set_header X-Real-IP $remote_addr;
     proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
     proxy_set_header Via "nginx";
  }
  # 关闭日志
  access_log off;
}
```

上面的配置表示将linjingc.top的请求，转发到服务器的5601端口，同时使用最基本的用户名、密码来认证。

#### 配置登录用户名，密码

```
htpasswd -c /data/nginx/db/passwd.db cat
```

注意passwd.db的路径要跟nginx配置中的一致，最后的cat为用户名，可以随便改，输入完该命令后，系统会提示输入密码，搞定后passwd.db中就有加密后的密码了，有兴趣的可以cat看下。

**提示：htpasswd是apache自带的小工具，如果找不到该命令，尝试用yum install httpd安装**

#### 关掉kibana端口的外网访问

用nginx转发后，一定要记得配置iptables之类的防火墙，禁止外部直接访问5601端口，这样就只能通过nginx来访问了

## 最后需要注意的是如果elasticsearch也设置了权限

进入 kibana的配置文件 

开启这两个选项 将用户密码填入

```java
 elasticsearch.username: "cat"
 elasticsearch.password: "password"
```

