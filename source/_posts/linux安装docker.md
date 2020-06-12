---
title: linux安装docker
date: 2020-06-07 19:08:47
tags: [linux,docker]
---

# linux安装docker

 ## 安装步骤

###  centOS 6的安装

```java
1.  yum install -y epel-release      
    类似安装redis需要先安装gcc库

2.  yum install -y docker-io
     安装docker

3. 安装后的配置文件: /etc/sysconfig/docker 
   修改 other_args="--registry-mirror=阿里云加速地址"

4. 启动docker后台服务: service docker start

5. docker version验证
```

<!--more-->

### centOS 7的安装

[官网文档地址](https://docs.docker.com/engine/install/centos/)

```java
1. 卸载老版本
   sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
                  
2. sudo yum install -y yum-utils
    安装yum-utils软件包（提供yum-config-manager 实用程序）并设置稳定的存储库
  
3. sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo  
   修改为:
    https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo 
   添加源地址 这里可以修改为用国内的地址 国外版本可能连接不上

4.sudo yum makecache fast
     更新yum软件包索引 加快速度
     
5. sudo yum install docker-ce docker-ce-cli containerd.io
  安装docker
  
6.  sudo systemctl start docker
  启动docker
  
7. sudo docker run hello-world
  通过运行hello-world 映像来验证是否正确安装了Docker Engine 。
     
8. 需要配置国内加速器 比如aliyun
     8.1  mkdir -p /etc/docker
     8.2 vim /etc/docker/daemon.json 
           添加加速器,例如网易的
          {"registry-mirrors":["http://hub-mirror.c.163.com"]}   
     8.3 systemctl daemon-reload
     8.4 systemctl restart docker
     
  
9.  sudo usermod -aG docker your-user
  如果要使用Docker作为非root用户，则现在应考虑使用类似以下方式将用户添加到“ docker”组 
  
```

#### centos7安装需要注意前置问题

```java
1.确认系统版本
  cat /etc/redhat-release

2. 需要添加依赖  
   lsb_release -a
   
3. yum安装gcc相关的依赖       查看gcc版本命令:gcc -v
    yum -y -install gcc
    yum -y -install gcc-c++
  
4.  需要添加国内加速器 不然下载镜像容易失败
```



## 卸载docker

```java
卸载Docker 
卸载Docker Engine，CLI和Containerd软件包：
1. systemctl stop docker
   停止docker运行
  
1. $ sudo yum remove docker-ce docker-ce-cli containerd.io
主机上的映像，容器，卷或自定义配置文件不会自动删除。要删除所有图像，容器和卷：

2. $ sudo rm -rf /var/lib/docker
您必须手动删除所有已编辑的配置文件。
```

