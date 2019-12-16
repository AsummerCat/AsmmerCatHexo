---
title: linux打包解压
date: 2019-12-15 22:12:29
tags: [linux]
---

# tar  格式:.tar

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

# gzip 格式:.gz

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



# tar+gzip

## 打包并压缩

```
tar zcvf /home/zzz.tar.gz 1.py 2.py
```

## 解包并解压缩

```
tar zxvf /home.zzz.tar.gz
```



# tar+bzip2  格式:.bz2

## 打包并压缩

```
tar jcvf /home/zzz.tar.bz2 1.py 2.py
```

## 解包并解压缩

```
tar jxvf /home.zzz.tar.bz2
```



# zip 文件压缩解压

## 压缩 

zip 压缩的文件名 压缩的文件

```
zip myzip.zip *.py
```

## 解压缩

unzip -d  解压后的目录位置 解压的文件

```
unzip -d myzip myzip.zip
```

