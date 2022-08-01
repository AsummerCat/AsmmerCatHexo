---
title: docker构建jdk镜像
date: 2018-12-04 15:38:10
tags: [docker]

---

#   下载centos7镜像



```linux
docker search centos
```

* 第一个是官方的

```
centos                             The official build of CentOS.                   5003                [OK]
```

* pull到本地

```
docker pull centos:7
```

* 然后 docker images

 ```
  可以看到centos的基础镜像只有200M
 ```

  <!--more-->

---

#  编译Dockerfile

## 新建一个文件，这里命名为jdk8centos7file

```
vi jdk8centos7file
```

> 很多地方都是使用的Dockerfile这种固定名称，其实创建的时候可以通过 -f 来指定dockerfile

## jdk8centos7file中的内容

> 可参考  [**Dockerfile**](https://github.com/arun-gupta/docker-images/blob/master/oracle-jdk/Dockerfile)

```linux
FROM centos:7

MAINTAINER  Cat

ADD jdk-8u191-linux-x64.tar.gz /usr/local/
#ADD apache-tomcat-7.0.67.tar.gz /usr/local/

ENV JAVA_HOME /usr/local/jdk1.8.0_191
ENV CLASSPATH $JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
ENV PATH=$JAVA_HOME/bin:$PATH
#ENV CATALINA_HOME /usr/local/apache-tomcat-7.0.67
#ENV CATALINA_BASE /usr/local/apache-tomcat-7.0.67
#ENV PATH #$PATH:$JAVA_HOME/bin:$CATALINA_HOME/lib:$CATALINA_HOME/bin

#容器运行时监听的端口
#EXPOSE  8080
```

>1、这里使用的镜像是上面下载的centos镜像； 
>2、jdk拷贝到dockerfile同级目录，如果在其它目录拷贝的时候可能出现找不到目录错误； 
>3、使用ADD指令会直接对jdk-8u144-linux-x64.tar.gz进行解压缩，不用再单独的tar解压jdk了。

##   使用Dockerfile创建镜像

```linux
docker build -t jdk8u191:2018 . -f jdk8centos7file
```

执行上面指令后在本地镜像中就能查询到创建的jdk镜像了



>1、 -t 指定镜像的名称和tag； 
>2、 使用-f 指定要使用的dockerfile，如果不指定会寻找当前目录名为Dockerfile的文件 
>3、上面有个 **.** ,这个表示当前目录，必不可少的



### 表示构建成功



```
{17:18}~/github ➭ docker build -t jdk8u191:2018 . -f jdk8centos7file
Sending build context to Docker daemon  323.8MB
Step 1/6 : FROM centos:7
 ---> 75835a67d134
Step 2/6 : MAINTAINER  Cat
 ---> Running in 7c4eebbeb576
Removing intermediate container 7c4eebbeb576
 ---> afcb29e0b0e7
Step 3/6 : ADD jdk-8u191-linux-x64.tar.gz /usr/local/
 ---> f1a5b017b31a
Step 4/6 : ENV JAVA_HOME /usr/local/jdk1.8.0_191
 ---> Running in 8bb276363553
Removing intermediate container 8bb276363553
 ---> 0719df9e2dc2
Step 5/6 : ENV CLASSPATH $JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
 ---> Running in f7e2e9f24de9
Removing intermediate container f7e2e9f24de9
 ---> 3cc5068802a3
Step 6/6 : ENV PATH $PATH:$JAVA_HOME/bin`
 ---> Running in ca8ecc498171
Removing intermediate container ca8ecc498171
 ---> 6f3c7fae5eec
Successfully built 6f3c7fae5eec
Successfully tagged jdk8u191:2018
```



---

##  查看构建后的jdk镜像



```
REPOSITORY          TAG                 IMAGE ID            CREATED              SIZE
jdk8u191            2018                6f3c7fae5eec        About a minute ago   597MB
```

到这里构建jdk镜像就完成了 

---



##   运行创建的镜像

```
docker run -d -it jdk8u191:1 /bin/bash
```



### ==注意==

创建容器的时候一定要使用 -it /bin/bash这种方式，要不然jdk的容器起不来。

## 验证镜像中的jdk

![](/img/2018-12-4/dockerFileToJdk.png)

