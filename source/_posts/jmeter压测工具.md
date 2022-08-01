---
title: jmeter压测工具
date: 2020-01-19 15:30:49
tags: [java,压测]
---

# JMeter下载地址：在官网 http://jmeter.apache.org/ 

# 下载

 在官网下载 二进制的压缩包就可以了 

如果下载src的 是没有编译的

# 解压

解压下载的二进制包，使用cmd命令进入bin目录，使用jmeter.bat启动程序。（注意直接双击jmeter.bat无法启动时需要使用Window+R，输入cmd，然后进入bin目录如下）

启动之后会有两个窗口，一个cmd窗口，一个JMeter的 GUI

不要使用GUI运行压力测试，GUI仅用于压力测试的创建和调试；执行压力测试请不要使用GUI。使用下面的命令来执行测试：

## 命令行操作

```python
jmeter -n -t [jmx file] -l [results file] -e -o [Path to web report folder]
```

![命令行](/img/2020-01-15/8.png)

<!--more-->

```
例1：测试计划与结果，都在%JMeter_Home%\bin 目录

> jmeter -n -t test1.jmx -l result.jtl 

例2：指定日志路径的：

> jmeter -n -t test1.jmx -l report\01-result.csv -j report\01-log.log      

例3：默认分布式执行：

> jmeter -n -t test1.jmx -r -l report\01-result.csv -j report\01-log.log

例4：指定IP分布式执行：

> jmeter -n -t test1.jmx -R 192.168.10.25:1036 -l report\01-result.csv -j report\01-log.log

例5：生成测试报表

> jmeter -n -t 【Jmx脚本位置】-l 【中间文件result.jtl位置】-e -o 【报告指定文件夹】

> jmeter -n -t test1.jmx  -l  report\01-result.jtl  -e -o tableresult

例6：已有jtl结果文件，运行命令生成报告

> jmeter -g【已经存在的.jtl文件的路径】-o 【用于存放html报告的目录】

> jmeter -g result.jtl -o ResultReport 

 

注意：

1）-e -o之前，需要修改jmeter.properties，否则会报错；

2）-l 与-o 目录不一样，最后生成两个文件夹下。

3）命令中不写位置的话中间文件默认生成在bin下，下次执行不能覆盖，需要先删除result.jtl；报告指定文件夹同理，需要保证文件夹为空

模板为report-template，结果目录D:\apache-jmeter-3.2\bin\resulttable
```



## gui操作

### 设置中文

![设置中文](/img/2020-01-15/1.png)

### 创建测试组

![创建测试组](/img/2020-01-15/2.png)

#### 设置并发大小

![设置并发大小](/img/2020-01-15/3.png)

### 创建HTTP请求

![创建HTTP请求](/img/2020-01-15/4.png)

![编写HTTP请求](/img/2020-01-15/5.png)

### 察看结果树

![察看结果树](/img/2020-01-15/6.png)

### 聚合报告

![聚合报告](/img/2020-01-15/7.png)



# 传递参数

## post请求传递参数

![post请求传递参数](/img/2020-01-15/9.png)

## 传递json参数

![传递json参数](/img/2020-01-15/10.png)

### 创建json消息头

![创建json消息头](/img/2020-01-15/11.png)

# 这样就构建了一个完整的案例了



# jmeter压力测试报错:java.net.BindException: Address already in use: connect解决办法（亲测有效）

最近在用jmeter做压力测试时，发现一个问题，当线程持续上升到某个值时，报错：java.net.BindException: Address already in use: connect

原因：windows提供给TCP/IP链接的端口为 1024-5000，并且要四分钟来循环回收它们，就导致我们在短时间内跑大量的请求时将端口占满了，导致如上报错。

解决办法（在jmeter所在服务器操作）：

```
1.cmd中输入regedit命令打开注册表；

2.在 HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters右键Parameters；

3.添加一个新的DWORD，名字为MaxUserPort；

4.然后双击MaxUserPort，输入数值数据为65534，基数选择十进制；

5.完成以上操作，务必重启机器，问题解决，亲测有效；
```

