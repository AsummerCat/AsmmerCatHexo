---
title: java打包exe
date: 2023-02-16 17:21:38
tags: [java]
---

# 打包工具官网下载地址
exe4j

https://exe4j.apponic.com/download

下载完成需注册激活
exe4j首页的 右下角 
激活码: A-XVK258563F-1p4lv7mg7sav

<!--more-->

## 打包携带jdk环境

注意如果需要打包成exe,不使用客户的jdk环境,需要把jdk一起打包在exe的同级目录里

操作步骤:

    在第六步的 JRE里
    选择 Search sequence
    加入 同级目录里的 jdk文件夹 使用绝对路径 这样换环境也可执行
    比如 ./jdk1.8

完成
