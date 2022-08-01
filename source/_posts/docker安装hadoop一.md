---
title: docker安装hadoop一
date: 2022-08-01 14:37:20
tags: [大数据,hadoop,docker]
---
# docker安装hadoop

centos_8_jdk_8
<!--more-->


## 基础镜像安装

```
1. docker pull centos:8

2. docker run -d --name=centos_8_jdk_8 --privileged centos:8 /usr/sbin/init

3. 进入容器
docker exec -it centos_8_jdk_8 bash

4.配置镜像

sed -e 's|^mirrorlist=|#mirrorlist=|g' \
         -e 's|^#baseurl=http://mirror.centos.org/$contentdir|baseurl=https://mirrors.ustc.edu.cn/centos|g' \
         -i.bak \
         /etc/yum.repos.d/CentOS-Linux-AppStream.repo \
         /etc/yum.repos.d/CentOS-Linux-BaseOS.repo \
         /etc/yum.repos.d/CentOS-Linux-Extras.repo \
         /etc/yum.repos.d/CentOS-Linux-PowerTools.repo \
         /etc/yum.repos.d/CentOS-Linux-Plus.repo
yum makecache

4.1 修复镜像

cd /etc/yum.repos.d

rm -f *.repo

curl -o /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-vault-8.5.2111.repo

yum makecache

修复完成

5. 安装 OpenJDK 8 和 SSH 服务：
yum install -y java-1.8.0-openjdk-devel openssh-clients openssh-server


6. 启用 SSH 服务：

systemctl enable sshd && systemctl start sshd


7.保存基础镜像
到这里为止，如果没有出现任何故障，一个包含 Java 运行环境和 SSH 环境的原型容器就被创建好了。这是一个非常关键的容器，建议大家在这里先在容器中用 exit 命令退出容器，然后运行以下下两条命令停止容器，并保存为一个名为 java_ssh 的镜像：

docker stop centos_8_jdk_8
docker commit centos_8_jdk_8 centos_8_jdk_8
```


# Hadoop 安装

```
1.创建hadoop单机容器
docker run -d --name=hadoop3_3 --privileged centos_8_jdk_8 /usr/sbin/init

2.hadoop压缩包拷贝到容器的root目录下
docker cp <你存放hadoop压缩包的路径> hadoop3_3:/root/

docker cp C:\Users\Administrator\Desktop\hadoop-3.3.3.tar.gz hadoop3_3:/root/


3.进入容器 ,进入/root目录
docker exec -it hadoop3_3 bash
cd /root

4.解压压缩包
tar -zxf hadoop-3.3.3.tar.gz

5.解压后将得到一个文件夹 hadoop-3.3.3，现在把它拷贝到一个常用的地方：

mv hadoop-3.3.3 /usr/local/hadoop

6.设置环境变量
echo "export HADOOP_HOME=/usr/local/hadoop" >> /etc/bashrc
echo "export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin" >> /etc/bashrc 

87. 找到java的home位置

which java
输出：/usr/bin/java
  
ls -lr /usr/bin/java
输出：/usr/bin/java -> /etc/alternatives/java
  
ls -lrt /etc/alternatives/java
输出：/etc/alternatives/java -> /usr/lib/jvm//usr/lib/jvm/java-1.8.0-openjdk-1.8.0.312.b07-2.el8_5.x86_64


8.配置java_home
vi /etc/profile

export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.312.b07-2.el8_5.x86_64

export JRE_HOME=$JAVA_HOME/jre 

export CLASSPATH=$JAVA_HOME/lib:$JRE_HOME/lib:$CLASSPATH 

export PATH=$JAVA_HOME/bin:$JRE_HOME/bin:$PATH

使其生效
source /etc/profile


9.退出容器再进去,验证
echo $HADOOP_HOME 的结果应该是 /usr/local/hadoop


10. 给hadoop配置环境变量

echo "export JAVA_HOME=/usr" >> $HADOOP_HOME/etc/hadoop/hadoop-env.sh
echo "export HADOOP_HOME=/usr/local/hadoop" >> $HADOOP_HOME/etc/hadoop/hadoop-env.sh

11.安装Hadoop后出现command not found，主要原因是没有配置路径
vi ~/.bashrc

export HADOOP_HOME=/usr/local/hadoop   #这里修改为自己的hadoop文件路径
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin

source ~/.bashrc
```

## 最后一步 需要添加相关配置
```

```
1.core-site.xml

```
java.lang.IllegalArgumentException: Invalid URI for NameNode address (check fs.defaultFS): file:/// has no authority.

添加:
<configuration>
 <property>
    <name>fs.defaultFS</name>
    <!--<value>hdfs://localhost:9000</value>-->
    <value>hdfs://172.17.0.3:9000</value>
 </property>
</configuration>


```
## 相关错误修复:

### 1.启动HDFS时报错ERROR: Attempting to operate on hdfs namenode as root
```
vi ~/.bashrc

# 文件中添加以下内容

export HDFS_NAMENODE_USER=root
export HDFS_DATANODE_USER=root
export HDFS_SECONDARYNAMENODE_USER=root
export YARN_RESOURCEMANAGER_USER=root
export YARN_NODEMANAGER_USER=root

source ~/.bashrc

```
### 2.修复确实秘钥的问题 Permission denied (publickey,gssapi-keyex,gssapi-with-mic,password)
就算本机启动也要配置ssh秘钥
```
ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
```

### 启动端口报错
```
hadoop配置文件的入口:
/usr/local/hadoop/etc/hadoop

```
### list of known
当主机使用ssh免密码登录时，弹出Warning:Permanently added (RSA) to the list of known hosts的警告，看着很碍眼。通过以下方法进行解决：
```
1：vim  /etc/ssh/ssh_config（master和slave1都需要设置）

找到#StrictHostKeyChecking ask去掉注释，并把ask改为no即可
```

## 启动命令
```
     start-all.sh
     stop-all.sh
     
     直接调用就可以了
```

```
NameNode format (格式化操作) 初次启动HDFS前
该操作 是进行HDFS的准备工作 创建工作目录等

命令: hdfs namenode -format
```



docker run -d --name=hadoop3_3open -P -p 8020:8020 -p 9000:9000 -p 9820:9820 -p 9870:9870 -p 19888:19888 -p 9864:9864 -p 8088:8088 hadoop3_3 /usr/sbin/init

yum install net-tools
ifconfig