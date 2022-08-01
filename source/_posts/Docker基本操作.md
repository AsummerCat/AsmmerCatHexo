---
title: Docker基本操作
date: 2018-10-01 17:21:31
tags: [docker]
---

>参考:  
>1.[菜鸟教程](http://www.runoob.com/docker/centos-docker-install.html)  
>2. [Docker语法大全](http://www.runoob.com/docker/docker-command-manual.html)  
>3.[Docker之容器的创建、启动、终止、删除、迁移等](https://www.dwhd.org/20151115_140935.html)  
>4.[如何在Docker容器内外互相拷贝数据？](https://blog.csdn.net/yangzhenping/article/details/43667785)  
>5.[如何在docker和宿主机之间复制文件](https://blog.csdn.net/xtfge0915/article/details/52169445)

<!--more-->

---



# 1.Docker镜像操作:

### 1.1列出镜像列表
>我们可以使用 docker images 来列出本地主机上的镜像

`docker images `

```
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
ubuntu              14.04               90d5884b1ee0        5 days ago          188 MB
php                 5.6                 f40e9e0f10c8        9 days ago          444.8 MB
nginx               latest              6f8d099c3adc        12 days ago         182.7 MB
mysql               5.6                 f2e8d6c772c0        3 weeks ago         324.6 MB
httpd               latest              02ef73cf1bc0        3 weeks ago         194.4 MB
ubuntu              15.10               4e3b13c8a266        4 weeks ago         136.3 MB
hello-world         latest              690ed74de00f        6 months ago        960 B
training/webapp     latest              6fae60ef3446        11 months ago       348.8 MB
```
各个选项说明:

REPOSITORY：表示镜像的仓库源

TAG：镜像的标签

IMAGE ID：镜像ID

CREATED：镜像创建时间

SIZE：镜像大小

---

同一仓库源可以有多个 TAG，代表这个仓库源的不同个版本，如ubuntu仓库源里，有15.10、14.04等多个不同的版本，我们使用 REPOSITORY:TAG   
来定义不同的镜像。

所以，我们如果要使用版本为15.10的ubuntu系统镜像来运行容器时，命令如下：

```
runoob@runoob:~$ docker run -t -i ubuntu:15.10 /bin/bash 
```

---

### 1.2查找镜像
>我们也可以使用 docker search 命令来搜索镜像。比如我们需要一个httpd的镜像来作为我们的web服务。我们可以通过 docker search 命令搜索 httpd 来寻找适合我们的镜像。

```
docker search httpd
```
NAME:镜像仓库源的名称

DESCRIPTION:镜像的描述

OFFICIAL:是否docker官方发布

---

### 1.3下载镜像

`docker pull 镜像名称`

---

### 1.4设置镜像标签

我们可以使用 docker tag 命令，为镜像添加一个新的标签。
>
docker tag 镜像ID，这里是 860c279d2fec ,用户名称、镜像源名(repository name)和新的标签名(tag)。

使用 docker images 命令可以看到，ID为860c279d2fec的镜像多一个标签。

```
runoob@runoob:~$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
runoob/centos       6.7                 860c279d2fec        5 hours ago         190.6 MB
runoob/centos       dev                 860c279d2fec        5 hours ago         190.6 MB
runoob/ubuntu       v2                  70bf1840fd7c        22 hours ago        158.5 MB
```

---

### 1.5 删除镜像
```
docker rmi REPOSITORY:TAG
```
---

# 2.Docker容器操作

###  2.1运行一个web应用
```
runoob@runoob:~# docker run -d -P training/webapp python app.py
```
<!--more-->

>参数说明:  
-d:让容器在后台运行。  
-P:将容器内部使用的网络端口映射到我们使用的主机上。

我们也可以通过 -p 参数来设置不一样的端口：

`runoob@runoob:~$ docker run -d -p 5000:5000 training/webapp python app.py`

---

### 2.2进入一个容器
`docker attach d48b21a7e439`
#### 2.2.1进入正在运行容器并以命令行交互
`docker exec -it e9410ee182bd /bin/sh`

#### 2.2.2以交互的方式运行
`docker run -i -t -p 8081:8080 tomcat:7 /bin/bash`

#### 2.2.3退出交互模式
   在容器中运行:
   `exit`

#### 2.2.4 进入交互模式后 无vim  

安装vim 执行两条命令

```
 1.apt update
 
 2.apt install vim
```

---


### 2.3 查看容器日志

 ` docker logs tomcat(容器名称)`


### 2.4 容器命名
#### 2.4.1 直接命名:
>当我们创建一个容器的时候，docker会自动对它进行命名。另外，我们也可以使用--name标识来命名容器，例如：

```
runoob@runoob:~$  docker run -d -P --name 别名 training/webapp python app.py  

43780a6eabaaf14e590b6e849235c75f3012995403f97749775e38436db9a441
```

---

#### 2.4.2 容器重命名

`docker rename oldName newName`

---


### 2.5 查看 WEB 应用容器

`docker ps  运行中的容器`  
`docker ps -a   全部容器`   
`docker ps -l   最近容器`  
`docker ps -s   容器大小`

---

### 2.6启动,重启,关闭 WEB 应用容器

启动:

```
runoob@runoob:~$ docker start wizardly_chandrasekhar
```

关闭:

```
runoob@runoob:~$ docker stop wizardly_chandrasekhar   

```
重启:

```
runoob@runoob:~$ docker restart wizardly_chandrasekhar   
```

>可以根据 CONTAINER ID 操作 和 name操作

---


### 2.7 移除WEB应用容器

`docker rm wizardly_chandrasekhar `  

删除容器时，容器必须是停止状态，否则会报如下错误:
>Error response from daemon: You cannot remove a running container  bf08b7f2cd897b5964943134aa6d373e355c286db9b9885b1f60b6e8f82b2b85.  
> Stop the container before attempting removal or force remove

---

### 2.8如何在docker和宿主机之间复制文件

```
从主机复制到容器:  
sudo docker cp host_path containerID:container_path

从容器复制到主机:  
sudo docker cp containerID:container_path host_path

容器ID的查询方法想必大家都清楚:docker ps -a

```

### 2.9 提交容器 成为镜像

```
docker commit -m="提交的描述信息" -a="作者" 容器ID 镜像名称:标签名
```

### 3.0  查看一个容器里面的进程信息

```
docker top 3b307a09d20d
UID      PID    PPID    C    STIME  TTY    TIME       CMD
root     805    787     0    Jul13   ?   00:00:00  nginx: master process nginx -g daemon off;
systemd+ 941     805     0   Jul13    ?   00:03:18  nginx: worker process
```

### 3.1 docker load && docker save

加载镜像和打包镜像

```
docker save 可以把一个镜像保存到 tar 文件中，你可以这么做：
~ docker save registry:2.7.1 >registry-2.7.1.tar
#同时 docker load 可以把镜像从 tar 文件导入到 docker 中
~ docker load < registry-2.7.1.tar
```

### 3.2 实时获取docker当前执行的事件

```
docker events
```

### 3.3 更新容器启动时候的参数 动态修改

`docker update`

当你 docker run 了之后却发现里面有一些参数并不是你想要的状态比如你设置的 nginx 容器 cpu 或者内存太小，这个时候你就可以使用 docker update 去修改这些参数。

```
~ docker update nginx --cpus 2
```

### 3.4 查看一个镜像是怎么构建的

```
 docker history
```

### 3.5 查看容器的退出状态

```
 docker wait 7f7f0522a7d0
 
 0显示正常退出
```

### 3.6 暂停容器运行

`docker pause && docker unpause`

```
当你运行了一个容器但是想要暂停它运行的时候，你就可以使用这个命令。

docker pause 7f7f0522a7d0
```

### 3.7  查看你修改了容器中的哪些文件

```
docker diff 38c59255bf6e
```

### 3.8 查看所有容器和占用内存和cpu使用的情况

`docker stats`

```
~ docker stats

CONTAINER ID        NAME                        CPU %               MEM USAGE / LIMIT     MEM %               NET I/O             BLOCK I/O           PIDS
1c5ade04e7f9        redis                        0.08%               17.53MiB / 47.01GiB   0.04%               10.9GB / 37GB       0B / 0B             4
afe6d4ebe409        kafka-exporter                0.09%               16.91MiB / 47.01GiB   0.04%               1.97GB / 1.53GB     752MB / 0B          23
f0c7c01a9c34        kafka-docker_zookeeper         0.01%               308.8MiB / 47.01GiB   0.64%               20.2MB / 12.2MB     971MB / 3.29MB      28
da8c5008955f        kafka-docker_kafka-manager     0.08%               393.2MiB / 47.01GiB   0.82%               1.56MB / 2.61MB     1.14GB / 0B         60
c8d51c583c49        kafka-docker_kafka            1.63%               1.256GiB / 47.01GiB   2.67%               30.4GB / 48.9GB     22.3GB / 5.77GB     85
......
```







