---
title: docker构建jar镜像
date: 2018-12-04 15:25:29
tags: [docker]

---

# docker 构建jar 镜像

* 此刻我假设你已经把你的 通过 maven  自动化构建的 java demo 打包了，就是说生成了以 jar 或者 war 结尾的包文件了。  

* 假设你已经成功安装 docker 。　

　　如果上面的俩个假设为True的话，那您可以接着往下看。

---

#  Docker的部署实践

##  创建dockerfile

**注意**我提到的假设：你的jar包或者war包都已经打包成功，并且docker安装成功。

dockerfile 的内容如下：　

```docker
FROM azul/zulu-openjdk:8
VOLUME /home/work/springShiroDemo
ADD shiro.jar shiro.jar
RUN bash -c 'touch /shiro.jar'
ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","/shiro.jar"]
```

* FROM：基于哪个镜像

* VOLUME：可以将本地文件夹或者其他container的文件夹挂载到container中

* ADD：将文件<src>拷贝到container的文件系统对应的路径<dest>

* RUN：RUN命令将在当前image中执行任意合法命令并提交执行结果。命令执行提交后，就会自动执行Dockerfile中的下一个指令

* ENTRYPOINT：container启动时执行的命令，但是一个Dockerfile中只能有一条ENTRYPOINT命令，如果多条，则只执行最后一条

　　　关于Dockerfile的介绍，可以查看：

```
https://www.dwhd.org/20151202_113538.html?lan=cn&lan=cn
```

> 新创建的  Dockerfile 文件需要和  jar 的在一个文件夹下

----

# **Jar包的生成**

这个用maven build就可以了

---

# **生成镜像**

