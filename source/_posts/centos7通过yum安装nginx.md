---
title: centos7通过yum安装nginx
date: 2018-09-28 17:22:46
tags: [nginx,linux]
---
### YUM安装
`yum install -y nginx`

---

### 通过yum安装的时候提示下面的错误
```
[root@localhost yum.repos.d]# yum install nginx
已加载插件：fastestmirror, langpacks
Loading mirror speeds from cached hostfile
没有可用软件包 nginx。
错误：无须任何处理
```
然后再执行安装命令
<!--more-->

`yum install -y nginx`

---
安装之后，可以查看nginx的默认安装目录

>[root@localhost yum.repos.d]# whereis nginx
nginx: /usr/sbin/nginx /usr/lib64/nginx /etc/nginx /usr/share/nginx /usr/share/man/man8/nginx.8.gz
[root@localhost yum.repos.d]# pwd
/etc/yum.repos.d

---

### 以下是Nginx的默认路径：

```
(1) Nginx配置路径：/etc/nginx/

(2) PID目录：/var/run/nginx.pid

(3) 错误日志：/var/log/nginx/error.log

(4) 访问日志：/var/log/nginx/access.log

(5) 默认站点目录：/usr/share/nginx/html
```

事实上，只需知道Nginx配置路径，其他路径均可在/etc/nginx/nginx.conf 以及/etc/nginx/conf.d/default.conf 中查询到


---


### nginx相关的验证命令及启动命令

```
nginx   启动

nginx -t  测试命令

nginx -s reload 修改nginx.conf之后，可以重载

关闭命令: ps -ef|grep nginx

kill -QUIT 2072   (master)


kill -int  2072 强制关闭nginx


kill -hup 2072 修改配置文件 可以重新读取

kill -usr1 2072 重读日志,在按日志备份切割有用 (大意是: 创建一个新的工作进程 然后关闭旧的工作进程) 

kill -usr2 2072   平滑的升级

kill -winch 2072   优雅的关闭旧的进程 配合 -usr2使用
```