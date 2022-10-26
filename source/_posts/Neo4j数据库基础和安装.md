---
title: Neo4j数据库基础和安装
date: 2022-10-26 16:43:28
tags: [Neo4j]
---
# Neo4j数据库基础和安装

## 图数据库介绍
图数据库是基于图论实现的一种NoSQL数据库,其数据存储结构和数据查询方式都是以图论为基础的,图数据库主要用于存储更多的连接数据

https://neo4j.com
<!--more-->
安装后,可以访问数据浏览器
127.0.0.1:7474/browser/

## 安装
注意: 安装4.x以上版本 需要jdk11的环境
高版本或者低版本都不行

默认账号密码: neo4j / neo4j
### 命令
```
## 将neo4j作为控制台应用执行
# /bin/neo4j console

## 将neo4j作为服务使用安装
/bin/neo4j install-service

console: 直接启动neo4j服务器

# 安装/卸载/更新neo4j服务
install0service | uninstall-service | update-service

start/stop/restart/status: 启动/停止/重启/状态

-v输出更多信息

```

### docker安装
下载镜像
```
docker pull neo4j:3.5.29-community
```
7474 for http
7473 for https
7487 for Bolt

##### 启动容器
```
docker run -d --name container_name -p 27474:7474 -p 27687:7687 -v /home/neo4j/data:/data -v /home/neo4j/logs:/logs -v /home/neo4j/conf:/var/lib/neo4j/conf -v /home/neo4j/import:/var/lib/neo4j/import --env NEO4J_AUTH=neo4j/123456 94edbsesds

```
```
	-d --name container_name   //-d表示容器后台运行 --name指定容器名字
	-p 27474:7474 -p 27687:7687   //映射容器的端口号到宿主机的端口号；27474 为宿主机端口
	-v /home/neo4j/data:/data   //把容器内的数据目录挂载到宿主机的对应目录下
	-v /home/neo4j/logs:/logs   //挂载日志目录
	-v /home/neo4j/conf:/var/lib/neo4j/conf   //挂载配置目录
	-v /home/neo4j/import:/var/lib/neo4j/import   //挂载数据导入目录
	--env NEO4J_AUTH=neo4j/password   //设定数据库的名字的访问密码
	neo4j //指定使用的镜像

```
修改配置文件
```
// 进入容器配置目录挂载在宿主机的对应目录，我这里是/home/neo4j/conf
cd /home/neo4j/conf

// vim编辑器打开neo4j.conf
vim neo4j.conf

// 进行以下更改
//在文件配置末尾添加这一行
dbms.connectors.default_listen_address=0.0.0.0  //指定连接器的默认监听ip为0.0.0.0，即允许任何ip连接到数据库

//修改
dbms.connector.bolt.listen_address=0.0.0.0:7687  //取消注释并把对bolt请求的监听“地址:端口”改为“0.0.0.0:7687”
dbms.connector.http.listen_address=0.0.0.0:7474  //取消注释并把对http请求的监听“地址:端口”改为“0.0.0.0:7474”

```
重启neo4j容器

## 1