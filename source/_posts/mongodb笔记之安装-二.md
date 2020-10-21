---
title: mongodb笔记之安装(二)
date: 2020-10-21 09:18:17
tags: [mongodb笔记]
---

# 安装

## 下载MongoDB数据库
打开官网：https://www.mongodb.com/download-center?jmp=nav#community
选择 Community Server 4.0.1 的版本。

## 安装步骤
在 D 盘创建安装目录，`D:\MongoDB`，将解压后的文件拷入新建的文件。  
在 D 盘创建一个目录，`D:\MongoDB\Server\4.0\Data`，用于存放 MongoDB 的数据。  
执行安装，使用命令行，进入 MongDB   的安装目录，执行安装命令，并指明存放 MongoDB 的路径  

<!--more-->

## 配置环境变量
`D:\MongoDB\Server\4.0\bin`

## 启动

linux运行的话 
```
启动:
mangod --path=xxx/data  --fork
关闭:
killall mongod
```
win:
```
命令: mongod.exe -dbpath=”D:\MongoDB\Server\4.0\data”。

最后一行显示我们的 MongoDB 已经连接到 27017,它是默认的数据库的端口；它建立完数据库之后，会
在我们的 MongoDbData 文件夹下，生成一些文件夹和文件：在 journal 文件夹中会存储相应的数据文
件，NoSQL 的 MongoDB，它以文件的形式，也就是说被二进制码转换过的 json 形式来存储所有的数据
模型。


# 创建启动脚本
启动 MongoDB 数据库，也可以根据自己配置 mongodb.bat 文件，在 D:\MongoDB\Server\4.0\bin 中
创建一个 mongodb.bat 文件，然后我们来编写这个可执行文件如下：
mongod --dbpath=D:\MongoDB\Server\4.0\data
运行 mongodb.bat 文件，MongoDB 便启动成功！

```


## linux下启动
也可以选中使用配置文件的方式  

1.新建一个mongodb.cfg的文件
```
dbpath=/opt/mogodb/data
logpath=/opt/mogodb/mongodb.log
fork=true
logappend=true
bind_ip=0.0.0.0

```
2. 启动: `mongod -f mongodb.cfg`

# 安装RoboMongo客户端
```
这样就可以直接访问 客户端了 就跟mysql数据库一样
```