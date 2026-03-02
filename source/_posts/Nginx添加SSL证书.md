---
title: nginx添加SSL证书
date: 2021-03-02 16:19:11
tags: [nginx]
---

```
server {
        listen 443 ssl;  # 1.1版本后这样写
        server_name www.domain.com; #填写绑定证书的域名
        ssl_certificate 1_www.domain.com_bundle.crt;  # 指定证书的位置，绝对路径
        ssl_certificate_key 2_www.domain.com.key;  # 绝对路径，同上
        ssl_session_timeout 5m;
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2; #按照这个协议配置
        ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE;#按照这个套件配置
        ssl_prefer_server_ciphers on;
        location / {
            root   html; #站点目录，绝对路径
            index  index.html index.htm;
        }
    }
```