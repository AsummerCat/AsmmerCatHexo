---
title: docker安装mongodb及其使用
date: 2019-11-22 13:33:02
tags: [docker,mongodb]
---

# docker安装mongodb及其使用

##  执行

```java
docker pull mongo （拉取镜像 默认最新版本）

docker images （查看镜像）

docker run -p 27017:27017 -td mongo （启动镜像）

docker ps （查看启动的镜像）

docker exec -it 镜像id /bin/bash （进入容器）

mongo （进入mongodb）
```

## 创建管理账户，然后退出。

```
use admin
db.createUser(
{
user: "admin",
pwd: "password",
roles: [ { role: "root", db: "admin" } ]
}
);
```

<!--more-->

##  （以刚建立的用户登录数据库 创建test用户）

```
mongo --port 27017 -u admin -p password --authenticationDatabase admin 
```

```java
use test
db.createUser(
{
user: "tester",
pwd: "password",
roles: [
{ role: "readWrite", db: "test" }
]
}
);

exit
```

##  (以刚创建的test用户登录)

```
mongo -u tester -p --authenticationDatabase test
```

## mongodb其他操作

```
mongodb其他操作命令

show dbs （显示数据库）

use  dbname (切换到数据库)

show collections （显示表）

db.find.表名 （查看表数据）
```

