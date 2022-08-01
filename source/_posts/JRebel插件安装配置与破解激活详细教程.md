---
title: JRebel插件安装配置与破解激活详细教程
date: 2019-03-05 20:01:53
tags: [前端]
---

# JRebel插件安装配置与破解激活（多方案）详细教程

[JRebel插件安装配置与破解激活（多方案）详细教程](https://www.cnblogs.com/wang1024/p/7211194.html)

## 操作

```
docker pull ilanyu/golang-reverseproxy
docker run -d -p 8888:8888 ilanyu/golang-reverseproxy
```



<!--more-->

下面是新的jrebel破解方式，采用本地License Server破解，该方法可破解最新版的jrebel（未来的某一天可能会失效，想要更稳定的破解方式请看后面的破解方法）

准备工作：下载反向代理软件（根据自己的系统下载对应版本，大多数人需要的都是ReverseProxy_windows_amd64.exe这个版本）

默认反代 idea.lanyus.com, 运行起来后，http://127.0.0.1:8888/Zephyr就是激活地址了（激活地址复制到激活的窗口，而不是浏览器地址栏，见下图）, 邮箱随意填写（激活成功前不要关闭反向代理程序）。

　　如果使用上面的激活地址出现  “Incorrect license server group URL.Contact license sever administrator.”  错误，是由于授权地址增加了GUID检测造成的，可以尝试使用下面的激活地址：

http://127.0.0.1:8888/88414687-3b91-4286-89ba-2dc813b107ce、

http://127.0.0.1:8888/ff47a3ac-c11e-4cb2-836b-9b2b26101696、

http://127.0.0.1:8888/11d221d1-5cf0-4557-b023-4b4adfeeb36a

点击Change license，显示已激活，完成！！

激活后一定要手动切换到离线模式，可离线180天，可随时重新点下“Renew Offline Seat”刷新激活周期，180天后激活状态会重新刷新