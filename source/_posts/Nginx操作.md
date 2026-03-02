---
title: nginx操作
date: 2018-09-29 17:48:39
tags: [nginx]
---


# 配置:

>主要由6个部分组成：
>main:用于进行nginx全局信息的配置  
events：用于nginx工作模式的配置  
http：用于进行http协议信息的一些配置  
server：用于进行服务器访问信息的配置  
location：用于进行访问路由的配置  
upstream：用于进行负载均衡的配置   


>参考配置详情地址:  

>https://blog.csdn.net/tsummerb/article/details/79248015

 


<!--more-->

# 请求转发

假设服务器域名为example.com，则对应的 nginx http配置如下：

```

http {
    server {
            server_name example.com;

            location /mail/ {
                    proxy_pass http://example.com:protmail/;
            }

            location /com/ {
                    proxy_pass http://example.com:portcom/main/;
            }

            location / {
                    proxy_pass http://example.com:portdefault;
            }
    }
}
```
以上的配置会按以下规则转发请求( GET 和 POST 请求都会转发):
可以用来转发 图片路径~~~之类的

```
将 http://example.com/mail/ 下的请求转发到 http://example.com:portmail/
将 http://example.com/com/ 下的请求转发到 http://example.com:portcom/main/
将其它所有请求转发到 http://example.com:portdefault/
```
>**<font style="color:black">提示: 必须把  location / {} 放在最下面不然访问不到 指定后缀</font>**

---
---

>可参考: 
>1.[Nginx实现负载均衡的几种方式](https://blog.csdn.net/qq_28602957/article/details/61615876)

# 负载均衡
>nginx支持的负载均衡调度算法方式如下：

>weight轮询（默认）：接收到的请求按照顺序逐一分配到不同的后端服务器，即使在使用过程中，某一台后端服务器宕机，nginx会自动将该服务器剔除出队列，请求受理情况不会受到任何影响。 这种方式下，可以给不同的后端服务器设置一个权重值（weight），用于调整不同的服务器上请求的分配率；权重数据越大，被分配到请求的几率越大；该权重值，主要是针对实际工作环境中不同的后端服务器硬件配置进行调整的。

>ip_hash：每个请求按照发起客户端的ip的hash结果进行匹配，这样的算法下一个固定ip地址的客户端总会访问到同一个后端服务器，这也在一定程度上解决了集群部署环境下session共享的问题。

>fair：智能调整调度算法，动态的根据后端服务器的请求处理到响应的时间进行均衡分配，响应时间短处理效率高的服务器分配到请求的概率高，响应时间长处理效率低的服务器分配到的请求少；结合了前两者的优点的一种调度算法。但是需要注意的是nginx默认不支持fair算法，如果要使用这种调度算法，请安装upstream_fair模块

>url_hash：按照访问的url的hash结果分配请求，每个请求的url会指向后端固定的某个服务器，可以在nginx作为静态服务器的情况下提高缓存效率。同样要注意nginx默认不支持这种调度算法，要使用的话需要安装nginx的hash软件包



upstream模块
upstream模块主要负责负载均衡的配置，通过默认的轮询调度方式来分发请求到后端服务器
简单的配置方式如下

```
upstream name {
    ip_hash;  
    server 192.168.1.100:8000;  
    server 192.168.1.100:8001 down;  
    server 192.168.1.100:8002 max_fails=3;  
    server 192.168.1.100:8003 fail_timeout=20s;  
    server 192.168.1.100:8004 max_fails=3 fail_timeout=20s;
}
```

核心配置信息如下

ip_hash：指定请求调度算法，默认是weight权重轮询调度，可以指定

server host:port：分发服务器的列表配置

-- down：表示该主机暂停服务

-- max_fails：表示失败最大次数，超过失败最大次数暂停服务

-- fail_timeout：表示如果请求受理失败，暂停指定的时间之后重新发起请求



# 静态页面分离

```


    #对jsp和do结尾的url也去访问tomcat服务
    location ~ \.(jsp|do)$ {  
            proxy_pass http://localhost:8080;
    }
    
    #对js、css、png、gif结尾的都去访问根目录下查找
    location ~ \.（js|css|png|gif）$ {  
            root F:/javaweb;
    }
```



# 完整配置文件

```
#user  nobody;
worker_processes  2;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;


events {
    #nginx默认最大并发数是1024个用户线程
    worker_connections  1024;
}


http {
    include       mime.types;
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    #http1.1在请求完之后还会保留一段时间的连接，所以这里的timeout时长不能太大，也不能太小，
    #太小每次都要建立连接，太大会浪费系统资源（用户不再请求服务器）
    keepalive_timeout  65;

    #gzip  on;

    server {
    #nginx监听80端口
        listen       80;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;
    #这里的/表示所有的请求
        #location / {
        #将80端口的所有请求都转发到8080端口去处理，proxy_pass代表的是代理路径
     #   proxy_pass http://localhost:8080;
          #  root   html;
           # index  index.html index.htm;
        #}

    #对项目名进行访问就去访问tomcat服务
    location  /Student_Vote {  
            proxy_pass http://localhost:8080;
    }

    #对jsp和do结尾的url也去访问tomcat服务
    location ~ \.(jsp|do)$ {  
            proxy_pass http://localhost:8080;
    }
    
    #对js、css、png、gif结尾的都去访问根目录下查找
    location ~ \.（js|css|png|gif）$ {  
            root F:/javaweb;
    }


        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #
        #location ~ \.php$ {
        #    proxy_pass   http://127.0.0.1;
        #}

        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        #location ~ \.php$ {
        #    root           html;
        #    fastcgi_pass   127.0.0.1:9000;
        #    fastcgi_index  index.php;
        #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
        #    include        fastcgi_params;
        #}

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #    deny  all;
        #}
    }


    # another virtual host using mix of IP-, name-, and port-based configuration
    #
    #server {
    #    listen       8000;
    #    listen       somename:8080;
    #    server_name  somename  alias  another.alias;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}


    # HTTPS server
    #
    #server {
    #    listen       443 ssl;
    #    server_name  localhost;

    #    ssl_certificate      cert.pem;
    #    ssl_certificate_key  cert.key;

    #    ssl_session_cache    shared:SSL:1m;
    #    ssl_session_timeout  5m;

    #    ssl_ciphers  HIGH:!aNULL:!MD5;
    #    ssl_prefer_server_ciphers  on;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}

}
```

