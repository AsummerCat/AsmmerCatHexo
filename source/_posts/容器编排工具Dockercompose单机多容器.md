---
title: 容器编排工具Dockercompose单机多容器
date: 2020-06-12 14:43:42
tags: [容器编排工具,docker]
---

# 容器编排工具Docker-compose单机多容器

## 介绍安装
自动化安装容器
彼此形成依赖关系

比如: 先安装mysql容器->再安装tomcat容器->安装nginx容器 ,其配置文件还可以绑定

只需要一个脚本就可以搞定了
通过yml文件定义多容器如何部署

win/mac默认提供Docker Compose, linux需安装

```
注意: Docker-compose是单机多容器的部署工具
如果需要集群的话 : 可以用k8s
```




<!--more-->

## 安装
[官方文档](docs.docker.com/compose)




## 使用
1.创建一个docker-compose.yml的配置文件
```
version: '3.3'

services:
  # 服务名
   db:
   # build: ./xxx/
   # 这句话意思是:构建xxx目录下的dockerfile文件
   
   # 这边是直接使用镜像 跟build二选一
     image: mysql:5.7
     volumes:
       - db_data:/var/lib/mysql
   # docker发现容器宕机 会自动重启       
     restart: always
   # 环境变量
     environment:
       MYSQL_ROOT_PASSWORD: somewordpress
       MYSQL_DATABASE: wordpress
       MYSQL_USER: wordpress
       MYSQL_PASSWORD: wordpress

   wordpress:
   # 设置依赖->底层依赖哪些容器
   # (docker层面上网络互联互通直接使用容器名可通信)
     depends_on:
       - db
     image: wordpress:latest
     ports:
       - "8000:80"
     restart: always
     environment:
       WORDPRESS_DB_HOST: db:3306
       WORDPRESS_DB_USER: wordpress
       WORDPRESS_DB_PASSWORD: wordpress
       WORDPRESS_DB_NAME: wordpress
volumes:
    db_data: {}
```

2. 构建->自动化部署
进入项目文件夹 进行构建
up 表示: 解析自动部署
-d 表示: 后台执行
然后进行等待自动化安装部署
```
docker-compose up -d
```

## 查看日志

在当前文件夹下
```
docker-compose logs
```

## 移除容器
在当前文件夹下
```
docker-compse down
```