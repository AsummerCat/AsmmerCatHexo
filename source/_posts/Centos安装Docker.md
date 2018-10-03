---
title: CentOS下YUM安装Docker
date: 2018-09-28 17:51:20
tags: [linux,docker]
---
### 1.安装yum环境
>sudo yum install -y yum-utils device-mapper-persistent-data lvm2

<!--more--> 

### 2.移除旧的版本
>$ sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-selinux \
                  docker-engine-selinux \
                  docker-engine
                  
### 3.安装一些必要的系统工具
>sudo yum install -y yum-utils device-mapper-persistent-data lvm2

### 4.添加软件源信息
>sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo


### 5.更新 yum 缓存
>sudo yum makecache fast

### 6.安装 Docker-ce
>sudo yum -y install docker-ce

### 7.启动 Docker 后台服务
>sudo systemctl start docker

### 8.测试运行 hello-world
>docker run hello-world

### 9.镜像加速
>鉴于国内网络问题，后续拉取 Docker 镜像十分缓慢，我们可以需要配置加速器来解决，我使用的是网易的镜像地址：http://hub-mirror.c.163.com。

新版的 Docker 使用 /etc/docker/daemon.json（Linux） 或者 %programdata%\docker\config\daemon.json（Windows） 来配置 Daemon。

请在该配置文件中加入（没有该文件的话，请先建一个）：

```
{
  "registry-mirrors": ["http://hub-mirror.c.163.com"]
}
```

### 10.卸载

```
删除 Docker CE
执行以下命令来删除 Docker CE：

$ sudo yum remove docker-ce
$ sudo rm -rf /var/lib/docker
```