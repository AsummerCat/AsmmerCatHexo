---
title: hadoop的安装部署二
date: 2022-08-01 14:37:55
tags: [大数据,hadoop]
---
# hadoop的安装部署二

## 下载
https://hadoop.apache.org/releases.html

## xsync命令

可同步运行多台机子相同的命令
https://blog.csdn.net/nalw2012/article/details/98322637

## 第一步 修改host 做映射
集群内的所有机子做映射

## 第二步 关闭防火墙 机器生成秘钥 免登

1.关闭防火墙
```
systemctl stop firewalld.service #关闭防火墙
systemctl disable firewalld.service # 进制防火墙开启自启

```
<!--more-->

2.ssh免密登录 (由一台主机生成三个秘钥 然后分配给其他机子)
```
ssh-keygen #4个回车 生成公钥和秘钥
ssh-copy-id node1    
ssh-copy-id node2
ssh-copy-id node3  
```
node1 2 3 标识映射的服务器host;


## 第三步 分布式环境下时间同步 及其jdk安装
1.集群同步时间(每台机子都要操作)

```
 yum -y install ntpdate
 
 ntpdate ntp4.aliyun.com
```

2. jdk1.8安装


## 目录结构
```
bin:      管理脚本的基础实现, 使用的话->用sbin目录下的脚本

etc:  配置文件所在目录

include: 头文件,通常用于C++访问HDFS或者编写MapReduce程序

lib:  动态库和静态库

libexec: 各个服务对用的shell配置文件所在的目录,
可用于配置日志输出,启动参数 JVM等基本信息修改

sbin: 管理脚本所在目录 ,
主要用于HDFS和YARN中各类服务的启动/关闭脚本

share: 各个模块编译后的jar包所在目录

```

## 编辑hadoop配置文件

1.hadoop-env.sh
```
cd hadoop-3.1.4/etc/hadoop/
vim hadoop-env.sh
```
```
#配置JAVA_HOME
export JAVA_HOME=jdk位置

# 设置用户以执行对应角色shell命令

export HDFS_NAMENODE_USER=root
export HDFS_DATANODE_USER=root
export HDFS_SECONDARYNAMENODE_USER=root
export YARN_RESOURCEMANAGER_USER=root
export YARN_NODEMANAGER_USER=root
```
2. core-site.xml 核心配置文件
```
<configuration>
<!--默认文件系统的名称.通过URI中schema区分不同文件系统.->
<!--file://本地文件系统 hdfa://hadoop分布式文件系统 gfs://.-->
<!--hdfs文件系统访问地址(NameNode地址): http:nn_host:8020  内部通信端口 8000 9000 or 9820也行-->
  <property>
    <name>fs.defaultFS</name>
    <value>hdfs://node1.itcast.cn:8020</value>
  <property>    
  <!--hadoop本地数据HDFS存储目录format时自动生成-->
  <property>
    <name>hadoop.tmp.dir</name>
    <value>/hadoop-3.1.4/data</value>
  <property>   
  <!--在web UI访问HDFS使用的用户名 .-->
  <property>
    <name>hadoop.http.staticuser.user</name>
    <value>root</value>
  <property>     
<configuration>
```

3.hdfs-site.xml HDFS的配置文件
```
<!--外部访问-namenode设定hdfs运行主机和端口-->
<property>
  <name>dfs.namenode.http-addrress</name>
  <value>node2.itcast.cn:9870</value>
</property>
<!--外部访问-secondaryNode设定hdfs运行主机和端口-->
<property>
  <name>dfs.namenode.secondary.http-addrress</name>
  <value>node2.itcast.cn:9868</value>
</property>

```

4.mapred-site.xml  MapReduce的配置文件
```
<!--mr程序默认运行方式 yarn集群模式,local 本地模式-->
<property>
  <name>mapreduce.framework.name</name>
  <value>yarn</value>
</property>
<!--MR APPMaster环境变量-->
<property>
  <name>yarn.app.mapreduce.am.env</name>
  <value>HADOOP_MAPRED_HOME=${HADOOP_HOME}</value>
</property>
<!--MR MapTask环境变量-->
<property>
  <name>mapreduce.map.env</name>
  <value>HADOOP_MAPRED_HOME=${HADOOP_HOME}</value>
</property>
<!--MR ReduceTask环境变量-->
<property>
  <name>mapreduce.reduce.env</name>
  <value>HADOOP_MAPRED_HOME=${HADOOP_HOME}</value>
</property>
```

5. yarn-site.xml yarn集群配置
```
<!-- yarn集群主角色RM运行机器 ResourceManager的地址-->
<property>
  <name>yarn.resourcemanager.hostname</name>
  <value>node1.itcast.cn</value>
</property>
<!--NodeManager上运行的附属服务.需配置成mapreduce_shuffle,才可运行MR程序,MR的协议-->
<property>
  <name>yarn.nodemanager.aux-services</name>
  <value>mapreduce_shuffle</value>
</property>
<!--每个容器请求的最小内存资源 MB为单位-->
<property>
  <name>yarn.scheduler.minimum-allocation-mb</name>
  <value>512</value>
</property>
<!--每个容器请求的最大内存资源 MB为单位-->
<property>
  <name>yarn.scheduler.maximum-allocation-mb</name>
  <value>2048</value>
</property>
<!--容器虚拟内存和物理内存之间的比率-->
<property>
  <name>yarn.nodemanager.vmem-pmem-ratio</name>
  <value>4</value>
</property>
```

6. workers 配置从角色的机器IP 配合一键启动使用
```
node1.itcast.cn
node2.itcast.cn
node3.itcast.cn
```
## 最后配置环境变量
```
vim /etc/profile

export HADOOP_HOME:安装路径
export PATH=$HADDOP_HOME/bin:$HADDOP_HOME/sbin

```
重新加载环境变量 验证生效
```
source /etc/profile
```



# NameNode format (格式化操作) 初次启动HDFS前
该操作 是进行HDFS的准备工作 创建工作目录等
```
命令: hdfs namenode -format
```


# 访问端口

## HDFS的NameNode
```
ip:9870
```

## yarn的ResourceManager
```
ip:8088
```

# 测试上传
```
hadoop fs -mkdir /input ;

hadoop fs -put /root/xxx.txt /input
```