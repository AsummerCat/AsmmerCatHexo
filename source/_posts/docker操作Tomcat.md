---
title: Docker操作Tomcat
date: 2018-10-02 15:11:16
tags: [docker]
---

# 1.在Docker查看是否已经安装Tomcat

`docker images`

---

<!--more-->

# 2.查询tomcat的镜像

`docker search tomcat`

---

# 3.下载tomcat镜像

`docker pull tomcat `

---

# 4.运行tomcat容器

```
runoob@runoob:~/tomcat$ docker run --name tomcat  -d -p 8080:8080 -v $PWD/test:/usr/local/tomcat/webapps/test  tomcat    

acb33fcb4beb8d7f1ebace6f50f5fc204b1dbe9d524881267aa715c61cf75320


runoob@runoob:~/tomcat$
```

命令说明：

-d 后台运行 
-p 8080:8080：将容器的8080端口映射到主机的8080端口

-v $PWD/test:/usr/local/tomcat/webapps/test：将主机中当前目录下的test挂载到容器的/test

---


# 5.进入交互模式:
#### 5.1进入正在运行容器并以命令行交互
`docker exec -it e9410ee182bd /bin/sh`

---

#### 5.2以交互的方式运行
`docker run -i -t -p 8081:8080 tomcat:7 /bin/bash`

---

#### 5.3 退出交互模式
   在容器中运行:
   `exit`

---

# 6.进入交互模式后没有vim 
 安装vim 执行两条命令
```
 1.apt update
 
 2.apt install vim
```
 
 ---

# 8.启动失败:

#### 8.1防火墙没打开  

`开启端口就可以了 如果是云服务器 打开安全组`

---

####  6.2 tomcat启动卡住

`参考另外一篇文章: linux安装tomcat8-5` 

---

# 9.查看日志

 `docker logs tomcat (容器)`

 