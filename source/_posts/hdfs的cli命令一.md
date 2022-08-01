---
title: hdfs的cli命令一
date: 2022-08-01 14:40:36
tags: [大数据,hadoop,hdfs]
---
# hadoop的fds的cli命令

# hdfs的基础操作

### <font color='blue'>拷贝HDFS文件 cp命令</font>
```
命令: hadoop fs -cp [-f]  <src> ...<dst>
-f 覆盖目标文件(如果存在的话)

```
<!--more-->

### <font color='blue'>追加数据到HDFS文件中  appendToFile命令</font>
```
命令: hadoop fs -appendToFile <localsrc> ...<dst>
将本地文件的内容追加到指定的dst人家.
如果dst文件不存在,则创建该文件
如果<localsrc>为-,则输入为从标准输入中读取.

```

### <font color='blue'>查看HDFS磁盘空间 df命令</font>
```
命令: hadoop fs -df [-h] <path>

显示文件系统的容量,可用空间和已用空间

path:  / 根目录
```

### <font color='blue'>查看HDFS文件使用的空间量 du命令</font>
```
命令: hadoop fs -du [-s] [-h] <path>

-s: 表示显示指定路径文件长度的汇总摘要,而不是单个文件的摘要
-h:  格式化文件大小

```

### <font color='blue'>HDFS数据移动操作  mv命令</font>
```
移动到HDFS指定文件夹下
可以使用该命令移动数据,重命名文件的名称

命令: hadoop fs -mv <src> ...<dst>

```

### <font color='blue'>HDFS数据复制操作  cp命令</font>
```
命令: hadoop fs -cp -f <src> ...<dst>

-f 覆盖目标文件(已存在的情况下)

```

### <font color='blue'>修改HDFS文件副本个数   setrep命令</font>
```
命令: hadoop fs -setrep [-R] [-W] <rep> <path>

修改指定文件的副本个数 (默认3副本)

-R 标识递归 修改文件夹下及其所有
-w 客户端是否等待副本修改完毕

<rep> 副本个数
```

# 目录相关内容

### <font color='blue'>创建目录 mkdir命令</font>
```
命令: hadoop fs -mkdir [-p] <path>

path 待创建的目录

-p 表示沿着路径创建父目录
```

### <font color='blue'>查看指定目录下的内容 ls命令</font>
```
命令: hadoop fs -ls [-h] [-R] <path>

path 指定目录路径

-h 显示文件size

-r 递归查看目录及其子目录

```

# 查看文件的相关命令

### <font color='blue'>查看HDFS文件内容 cat命令</font>
```
命令: hadoop fs -cat <src>

读取指定文件全部内容,显示标准输出控制台

src 表示 hdfs的路径

注意: 对于大文件内容读取,慎重

```

### <font color='blue'>查看HDFS文件内容 head命令</font>
```
命令: hadoop fs -head <file>
查看文件前1kb的内容

```

### <font color='blue'>查看HDFS文件内容 tail命令</font>
```
命令: hadoop fs -tail [-f] <file>
查看文件最后1kb的内容

-f 选择可以动态显示文件中追加的内容

```

# 文件上传下载相关命令

### <font color='blue'>上传文件到HDFS指定目录 put命令</font>
```
命令: hadoop fs -put [-f] [-p] <localsrc> ...<dst>

-f 覆盖目标文件(已存在情况下)
-p 保留访问和修改时间,所有权和权限
localsrc 本地文件系统 (客户端所在机器)
dst 目标文件系统(HDFS)


命令: hadoop fs -moveFromLocal <localsrc> ...<dst>
和-PUT功能相同 ,只不过上传结束后,源数据会被删除

```

### <font color='blue'>下载HDFS文件到本地  get命令</font>
```
命令: hadoop fs -get [-f] [-p] <src> ...<localsrc>

下载文件到本地系统指定目录,localdst必须是目录

-f 覆盖目标文件(如果已经存在该文件的话)
-p 保留访问和修改时间,所有权和权限

```

### <font color='blue'>合并下载HDFS文件到本地 getmerge命令</font>
```
命令: hadoop fs -getmerge [-nl] [-skip-empty-file] <src> <localdst>

下载多个文件合并到本地文件系统的一个文件中.

-nl 标识在每个文件末尾添加换行符

[-skip-empty-file] 跳过空文件

例如:

hadoop fs -getmerge /tmp/small/* ./6.txt
拷贝small目录下的所有文件合并到 6.txt
```

