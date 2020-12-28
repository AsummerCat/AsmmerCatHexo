---
title: shadowsocks安装
date: 2020-12-28 16:30:24
tags: [linux,shadowsocks]
---

# 安装服务端

## 安装相关内容

### 安装pip
```
curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py
```
然后，输入`python get-pip.py`之后回车

### 安装shadowsocks
```
pip install shadowsocks
```
<!--more-->
## 配置shadowsocks

```
vi /etc/shadowsocks.json

内容:

{
    "server":"0.0.0.0",
    "server_port":9999,
    "local_port":1080,
    "password":"1234567890",
    "timeout":600,
    "method":"aes-256-cfb"
}
```
### 将shadowsocks加入系统服务
```
输入编辑文件命令
vi /etc/systemd/system/shadowsocks.service并回车

内容:

[Unit]
Description=Shadowsocks
[Service]
TimeoutStartSec=0
ExecStart=/usr/bin/ssserver -c /etc/shadowsocks.json
[Install]
WantedBy=multi-user.target
```

### 启动shadowsocks服务并设置开机自启
```
#设置开机自启命令
systemctl enable shadowsocks

#启动命令
systemctl start shadowsocks

#查看状态命令
systemctl status shadowsocks

#关闭服务
systemctl stop shadowsocks
```

## 客户端
```
https://github.com/shadowsocks/shadowsocks-windows
```