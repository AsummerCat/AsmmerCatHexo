---
title: linux打包解压
date: 2019-12-15 22:12:29
tags: [linux]
---

## 打包

tar -cvf 包名 文件1 文件2

```linux
tar -cvf /home/zzz.tar 1.py 2.py
```

## 解包

tar -xvf 包名

```java
tar -xvf /home/zzz.tar
```

<!--more-->

## 压缩包

gzip xxx.tar

```
gzip -r 压缩子文件
gzip /home/zzz.tar
```

## 解压缩包

gzip -d xxx.tar

```
gzip -d /home/zzz.tar.gz
```

