---
title: linux安装hexo教程
date: 2023-12-06 15:34:31
tags: [hexo]
---
## 1.安装git 和nginx

`yum install -y nginx git`

<!--more-->

## 2. 安装nvm node管理
```
   wget -qO- https://raw.githubusercontent.com/creationix/nvm/v0.39.6/install.sh | bash
2.1、刷新配置即可正常使用
#刷新配置
source ~/.bashrc
#判断nvm是否安装
nvm -v
2.2、使用nvm下载相关node版本
nvm install 14.13.2

#nvm常用命令
nvm uninstall 14.13.2     // 移除 node 14.13.2
nvm use 14.13.2           // 使用 node 14.13.2
nvm ls                   // 查看目前已安装的 node 及当前所使用的 node
nvm ls-remote            // 查看目前线上所能安装的所有 node 版本
nvm alias default 14.13.2 // 使用 14.13.2 作为预设使用的 node 版本
```
nvm默认安装位置: `cd /root/.nvmcd`



## 3.下载对应hexo地址

hexo地址
`git clone https://github.com/AsummerCat/AsmmerCatHexo.git`
//文章地址
`git clone https://github.com/AsummerCat/AsummerCat.github.io.git`

进行npm install 使hexo命令可以使用
```
npm install -g hexo-cli
```


4.nginx 新增静态文件映射路径
```
/home/AsummerCat.github.io
```


5.新增ssl证书
```
server {
#HTTPS的默认访问端口443
#如果未在此处配置HTTPS的默认访问端口，可能会造成Nginx无法启动。
listen 443 ssl;
#填写证书绑定的域名
server_name linjingc.top;

        #填写证书文件绝对路径
        ssl_certificate      "/home/cert/linjingc.top.pem";
        #填写证书私钥文件绝对路径 
        ssl_certificate_key  "/home/cert/linjingc.top.key";
 
        ssl_session_cache    shared:SSL:1m;
        ssl_session_timeout  5m;
		
        #默认加密套件
        ssl_ciphers  HIGH:!aNULL:!MD5;
		
        #自定义设置使用的TLS协议的类型以及加密套件（以下为配置示例，请您自行评估是否需要配置）
        #TLS协议版本越高，HTTPS通信的安全性越高，但是相较于低版本TLS协议，高版本TLS协议对浏览器的兼容性较差。
        #ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
        #ssl_protocols TLSv1.1 TLSv1.2 TLSv1.3;
		
        #表示优先使用服务端加密套件。默认开启
        ssl_prefer_server_ciphers  on;
 
        location / {
            root   html;
            index  index.html index.htm;
        }
    }
}
```

