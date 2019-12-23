---
title: linux上利用scp拷贝文件
date: 2019-12-16 22:56:17
tags: [linux]
---

# scp控制台远程拷文件

如果在没有ftp图形化界面的情况下拷贝文件



```
scp 远程用户@远程服务器地址:远程路径 拷贝到的路径
```



## 测试语句

### 接收远程文件

```
scp root@192.168.1.1:/home/java/test.java /home/java/rei.java
```

### 发送远程文件

```
scp /home/java/rei.java root@192.168.1.1:/home/java/test.java
```



 