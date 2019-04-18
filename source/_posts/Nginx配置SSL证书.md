---
title: Nginx配置SSL证书
date: 2019-04-19 07:28:49
tags: nginx
---

# Nginx配置SSL证书

# 先申请个ssl免费证书再说

阿里云免费SSL证书  腾讯云免费SSL证书

# 开始配置ssl

## 配置SSL证书到Nginx中

将 .crt 和 .key 放入到一个文件夹下   

如果是linux系统，推荐放到/etc/ssl/目录下

<!--more-->

## 找到nginx的配置文件

`nginx.conf`

然后我们需要去找到nginx的配置文件。   
关于这个配置文件的路径查找，推荐大家启动nginx后使用ps -ef | grep nginx的命令去查找，这样可以很简单的找到真正会生效的那个配置文件。

## 在http节点中 添加server节点

```
http{

}
```

在这里加入  

```
http{
    #http节点中可以添加多个server节点
    server{
        #监听443端口
        listen 443;
        #对应的域名，把linjingc.top改成你们自己的域名就可以了
        server_name linjingc.top;
        ssl on;
        #从腾讯云获取到的第一个文件的全路径
        ssl_certificate /etc/ssl/1_baofeidyz.com_bundle.crt;
        #从腾讯云获取到的第二个文件的全路径
        ssl_certificate_key /etc/ssl/2_baofeidyz.com.key;
        ssl_session_timeout 5m;
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
        ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE;
        ssl_prefer_server_ciphers on;
        
        location / {
          # 负载均衡转发
               proxy_pass http://linuxTest; 

                #文件夹
                root /usr/local/service/ROOT;
                #主页文件
                index index.html;
        }
    }

}

```

这里还不行，因为如果用户使用的是http协议进行访问，那么默认打开的端口是80端口，所以我们需要做一个重定向，我们在上一个代码块的基础上增加一个server节点提供重定向服务。

```
server{
        listen 80;
        server_name linjingc.top;
        rewrite ^/(.*)$ https://linjingc.top:443/$1 permanent;
    }

```

## 重新加载配置文件

然后使用保存配置文件，使用`nginx -t`命令对文件对配置文件进行校验，如果看到successful表示文件格式证书，这时候我们就可以启动nginx服务或者重新加载nginx配置文件。 

```
启动nginx服务：service nginx start 
重新加载配置文件：nginx -s reload

```

## 报错处理

验证一下修改是否生效。`./sbin/nginx -t` 检测正确后重启`nginx ./sbin/nginx -s reload` 
但是报错 
`nginx:[emerg]unknown directive “ssl” `
因为安装的时候没有把nginx的ssl模块编译安装好。可以重新安装一下。