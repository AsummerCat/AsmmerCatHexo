---
title: docker安装jenkins
date: 2020-05-23 18:42:56
tags: [Jenkins,docker]
---

# docker安装jenkins

## 参考地址:https://www.jianshu.com/p/eabf80b7b0e6

## pull镜像

```
 docker pull jenkins:2.60.3
```

## 查看是否最新版本

```
docker inspect ba607c18aeb7
```

## 创建一个jenkins目录

```
mkdir /home/jenkins_home;
```

<!--more-->

## 启动容器

```
docker run -d --name jenkins_01 -p 8081:8080 -v /home/jenkins_01:/home/jenkins_01 jenkins/jenkins:lts
```

## 查看jenkins服务 

```
docker ps | grep jenkins;
```

## 启动服务端

```
localhost:8081
```

## 获取密码

```
进入容器内部   docker exec -it jenkins_01 /bin/sh
执行：cat /var/jenkins_home/secrets/initialAdminPassword，得到密码并粘贴过去
```

## 重启docker镜像

```
重启docker镜像 docker restart {CONTAINER ID}，安装完毕。
```

