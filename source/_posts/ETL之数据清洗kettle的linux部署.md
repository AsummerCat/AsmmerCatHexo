---
title: ETL之数据清洗kettle的linux部署
date: 2022-08-01 14:15:36
tags: [ETL工具,kettle,大数据]
---

# ETL之数据清洗kettle的linux部署

kettle用来做数据清洗,这边可以转换其他db输出

## 部署linux运行脚本

### Pan.sh 转换执行引擎  (执行单脚本)
自定义参数使用${xxx} 来表示
```
该shell脚本在 kettle的目录里
pan.sh 可以用来在服务器中执行一个转换

pan.sh的命令行参数:

-version: 显示版本信息

-file: 指定要运行的转换文件(XML文件)

-level: 设置日志级别(Basic,Detailed,Debug,Rowlevel,Error,Nothing)

-log: 指定日志文件

-param: key=value (该参数可以指定多个, 一个参数带一个-param)
```

<!--more-->
参考脚本
```
pam.sh -file /xxx.ktr -level Basic -param:a=xxx -param:b=xxx
```

### Kitchen.sh 作业执行引擎 (执行作业任务)
跟windows相同,需要添加驱动jar包,比如mysql的

```
需要在作业任务中,添加
${internal.Entry.Current.Directoy}/xxxx.ktr 
(默认使用的参数 作业执行的当前目录)

```
参数跟上面的脚本是差不多的
例子:
```
Kitchen.sh -file /xxx.kjb -level Basic -param:a=xxx ```