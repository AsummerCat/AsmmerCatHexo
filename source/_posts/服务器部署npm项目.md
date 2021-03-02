---
title: 服务器部署npm项目
date: 2021-03-02 16:18:24
tags: [npm,nginx]
---

# NPM打包
```
npm run build:prod
```


# nginx部署
`cd etc/nginx`
打开`nginx.conf`配置文件

修改
```
打包后的文件地址 :/home/admin-ui

 location / {
            root   /home/admin-ui;   #访问路径，相当于Tomcat的ROOT，这里自己配 
            index  index.html index.htm;   #访问index
        }
```