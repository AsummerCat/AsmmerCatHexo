---
title: linux安装Redis
date: 2024-05-26 22:56:22
tags: [redis,linux]
---
# 在CentOS或Fedora上安装

## 1.  打开终端。
## 2.  安装Redis：

```
sudo yum install redis
```
<!--more-->
在Fedora上，可能需要使用`dnf`而不是`yum`：

```
sudo dnf install redis
```
## 3.启动Redis服务：

```
 sudo systemctl start redis
```
## 4.检查Redis服务状态：

```
sudo systemctl status redis

```
## 5.要使Redis在启动时自动启动，可以使用：

`sudo systemctl enable redis`

## 6.卸载

```
sudo yum remove redis
sudo yum autoremove  # 删除所有不再需要的包
sudo rm -rf /etc/redis/
sudo rm -rf /var/lib/redis/
sudo rm -rf /var/log/redis/

```

## 7.修改redis密码

查看配置文件位置

`cat /lib/systemd/system/redis.service`

修改密码

`requirepass yournewpassword`

重启



## 8.移除限制访问 允许外部访问
修改配置文件

`# bind 127.0.0.1`
