---
title: windows安装aria2及其使用
date: 2019-12-31 11:51:21
tags: [aria2]
---

# windows安装aira2及其使用



Aria2是一个命令行下运行、多协议、多来源下载工具（HTTP/HTTPS、FTP、BitTorrent、Metalink），并且支持迅雷离线以及百度云等常用网盘的多线程下载（甚至可以超过专用客户端的下载速度）。

Aria2在Windows、Mac OS、Android、Linux均有相应的版本，在具体的配置过程有细微的区别。这里我是在Windows上配置aria2。

Aria2与传统的下载软件有较大的区别，它**没有图像用户界面**，并且安装配置aria2**实际上是在配置一个服务器**。

## 下载

aria2下载地址：
https://github.com/aria2/aria2/releases

<!--more-->

## 安装

解压后随便找个英文路径的丢进去就行了。
我就丢在D:\aria2\下。

接下来新建几个文件：

```Aria2.log （日志，空文件就行）
aria2.session （下载历史，空文件就行）
aria2.conf （配置文件）
HideRun.vbs （隐藏cmd窗口运行用到的）
```

## 配置

### 配置aria2.conf

完整配置文件: https://aria2c.com/usage.html

````
用文本编辑工具打开刚才建立的aria2.conf
复制按下面的内容，
注意修改一下选项：
dir=D:\td\ （下载文件保存路径，改为你想要的）
log=D:\aria2\Aria2.log （日志文件，如果不需要日志，这一行可去掉，如果需要，路径D:\aria2\改为你安装aria2的路径）
input-file=D:\aria2\aria2.session
save-session=D:\aria2\aria2.session（这两个是记录和读取下载历史用的，断电和重启时保证下载任务不会丢失，如果有时aria2不能启动，清空这里面的内容就行了，路径D:\aria2\改为你安装aria2的路径）
````

### see --split option -拆分选项

```
max-concurrent-downloads=5
continue=true
max-overall-download-limit=0
max-overall-upload-limit=50K
max-upload-limit=20
```

### Http/FTP options

```
connect-timeout=120
lowest-speed-limit=10K
max-connection-per-server=10
max-file-not-found=2
min-split-size=1M
split=5
check-certificate=false
http-no-cache=true
```

### BT/PT Setting   RPC选项

```
bt-enable-lpd=true
#bt-max-peers=55
follow-torrent=true
enable-dht6=false
bt-seed-unverified
rpc-save-upload-metadata=true
bt-hash-check-seed
bt-remove-unselected-file
bt-request-peer-speed-limit=100K
seed-ratio=0.0
```

### RPC Options

```
enable-rpc=true
pause=false
rpc-allow-origin-all=true
rpc-listen-all=true
rpc-save-upload-metadata=true
rpc-secure=false
```

### Advanced Options 高级选项

```
daemon=true
disable-ipv6=true
enable-mmap=true
file-allocation=falloc
max-download-result=120
#no-file-allocation-limit=32M
force-sequential=true
parameterized-uri=true
```

## 实现开机无cmd窗口启动

```
用文本编辑工具打开刚才建立的HideRun.vbs

复制以下内容，注意修改D:\aria2\ 为你的aria2安装路径(vbs路径不能超过8个字符，我用不好所以这个软件放在根目录了，其他目录长了就找不到文件)

CreateObject("WScript.Shell").Run "D:\aria2\aria2c.exe --conf-path=aria2.conf",0

要启动aria2，一定要点击这个文件，不要点击aria2c.exe

如果要开机启动，创建一个HideRun.vbs的快捷方式，把快捷方式丢到 C:\Users\用户名\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup中

我用的是windows8.1，windowsxp和7，自己找一下路径
```

点击HideRun.vbs启动服务器，**Windows系统会提示防火墙，点击允许外网访问该应用**。这个时候虽然看不到任何用户界面，但程序实际上已经在后台运行了，**用资源管理器可以看到正在运行的程序**。

## 管理界面

webui，再来控制服务器。

点击：https://github.com/ziahamza/webui-aria2下载

之后我们就可以选择链接、种子等下载相应文件了。

### [aria2c.com](http://aria2c.com/)

Aria2c.com 是一个基于 aria2 的网页版控制端，你可以通过这个网站来添加、删除、管理下载任务。

理论上，打开就能用，无需设置

如果看到右上角的 Aria2 版本号，那么就已经正常工作了。

# 启动

aria2c.exe --conf-path=config.conf

