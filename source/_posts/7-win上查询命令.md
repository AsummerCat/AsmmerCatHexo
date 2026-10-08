---
title: 7.win上查询命令
date: 2026-10-08 17:26:58
tags: [网络安全]
---
## 1. 查看本机用户信息
```
net user
```
<!--more-->

## 2. 查看本机用户信息详情
```
net user xxxx
```
## 3.获取本地管理员信息
```
net localgroup administrators
```
## 4. 查看当前在线用户
```
quser
```
## 5. 查当前用户在目标系统中的具体权限
```
whoami /all
```

## 6.查看当前权限
```
whoami && whoami/priv
```

## 7.查看本机所有TCP,UDP端口
```
netstat -ano
```

## 8. 查看arp缓存
```
arp -a
```

## 9.防火墙相关
```
关闭防火墙
windows Server 2003之前版本
netsh firewall set opmode disable

windows Server 2003之后版本
netsh advfirewall set allprofiles state off

查看防火墙配置
netsh firewall show config

查看配置规则
netsh advfirewall firewall show rule name=all

```
## 10 域相关
```
Builtin容器: 保存域中本地安全组
Computers容器: 是存放 windows server域内所有成员计算机的计算机账户
Domain Controllers容器: 用于保存当前域控制器下创建的所有子域和辅助域
Users容器: 主要用于保存系统自动创建的用户和登录到当前域控制器的所有用户账户


```s
