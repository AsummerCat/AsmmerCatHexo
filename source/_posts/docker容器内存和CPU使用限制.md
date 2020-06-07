---
title: docker容器内存和CPU使用限制
date: 2020-06-07 23:09:49
tags: [docker]
---

# docker容器内存和CPU使用限制

## 示例如下

```java
sudo docker run --name seckill0 -p 8080:8080 -m 1024M --cpus=0.2 -d seckill:v0
sudo docker run --name seckill1 -p 8081:8080 -m 1024M --cpus=0.2 -d seckill:v0
sudo docker run --name seckill2 -p 8082:8080 -m 1024M --cpus=0.2 -d seckill:v0
```

- -m:限制内存使用为1G

- --cpus：限制CPU使用的百分比，这里设置为100%

- –vm 1：启动 1 个内存工作线程。

- –vm-bytes 280M：每个线程分配 280M 内存。

  

查看容器的内存CPU等情况：docker stats contrainer id