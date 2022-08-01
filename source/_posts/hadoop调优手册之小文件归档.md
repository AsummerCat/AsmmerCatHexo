---
title: hadoop调优手册之小文件归档
date: 2022-08-01 16:32:07
tags: [大数据,hadoop,hdfs,hadoop调优手册]
---
# hadoop调优手册之小文件归档

## 归档操作,将小文件合并成一个.har的归档文件
这样可以减少文件数,让磁盘能存放更多的大文件

#### 操作
1.启动yarn进程
```
start-yarn.sh
```

2.归档文件整理
将A目录下的文件都归档为一个叫`input.har`的归档文件,并把归档文件存储到`/output`路径下

```
hadoop archive -archiveName input.har -p /input /output
```

3.查看归档
```
命令: hadoop fs -ls har://

hadoop fs -ls har:///output/input.har
```
<!--more-->

4.解压 归档文件
```
命令:hadoop fs -cp har://

hadoop fs -cp har:///output/input.har/*  /解压目录
```

