---
title: docker安装zookeeper
date: 2019-09-16 22:49:28
tags: [docker,zookeeper]
---

# docker安装zookeeper

## 下载Zookeeper镜像

`docker pull zookeeper`

## 启动容器并添加映射

` docker run --privileged=true -d --name zookeeper --publish 2181:2181  -d zookeeper:latest`

## 查看容器是否启动

`docker ps`

