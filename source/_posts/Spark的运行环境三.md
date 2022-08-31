---
title: Spark的运行环境三
date: 2022-08-31 14:04:10
tags: [大数据,Spark]
---
# Spark的运行环境

# Local 模式
```
所谓的 Local 模式，就是不需
要其他任何节点资源就可以在本地执行 Spark 代码的环境
```
idea中的那种 是开发环境

#### 运行Local模式
将 `spark-3.0.0-bin-hadoop3.2.tgz` 文件上传到 Linux 并解压缩，放置在指定位置
```
tar -zxvf spark-3.0.0-bin-hadoop3.2.tgz -C /opt/module
cd /opt/module 
mv spark-3.0.0-bin-hadoop3.2 spark-local
```
进入解压缩后的路径，执行如下指令
```
bin/spark-shell
```
启动成功后，可以输入网址进行 Web UI 监控页面访问
```
虚拟机ip:4040
```
<!--more-->
退出本地模式
```
按键 Ctrl+C 或输入 Scala 指令
:quit
```
#### 提交就业任务
```
bin/spark-submit \
--class org.apache.spark.examples.SparkPi \
--master local[2] \
./examples/jars/spark-examples_2.12-3.0.0.jar \
10
```
1) --class 表示要执行程序的主类，此处可以更换为咱们自己写的应用程序
2) --master local[2] 部署模式，默认为本地模式，数字表示分配的虚拟 CPU 核数量
3) spark-examples_2.12-3.0.0.jar 运行的应用类所在的 jar 包，实际使用时，可以设定为咱
   们自己打的 jar 包
4) 数字 10 表示程序的入口参数，用于设定当前应用的任务数量

# Standalone 模式
只使用 Spark 自身节点运行的集群模式，也就是我们所谓的
独立部署（Standalone）模式。
Spark 的 Standalone 模式体现了经典的 master-slave 模式。
```
集群模式下 master监控页面默认访问端口为8080

修改spark-env.sh 可以自定义
SPARK_MASTER_WEBUI_PORT=8989
```

![1](/img/2022-08-23/1.png)
![1](/img/2022-08-23/2.png)
![1](/img/2022-08-23/3.png)

# 任务参数详解
在提交应用中，一般会同时一些提交参数
```
bin/spark-submit \
--class <main-class>
--master <master-url> \
... # other options
<application-jar> \
[application-arguments]
```
| 参数 | 解释 | 可选值举例 |
| --- | --- | --- |
| --class |Spark 程序中包含主函数的类  |  |
|--master|Spark 程序运行的模式(环境)|模式：local[*]、spark://linux1:7077、Yarn|
|--executor-memory 1G|指定每个 executor 可用内存为 1G|符合集群内存配置即可，具体情况具体分析。|
|--total-executor-cores 2|指定所有executor使用的cpu核数为 2 个||
|--executor-cores|指定每个executor使用的cpu核数||
|application-jar|打包好的应用 jar，包含依赖。这个 URL 在集群中全局可见。 比 如hdfs://共享存储系统，如果是file://path，那么所有的节点的path 都包含同样的 jar||
|application-arguments|传给 main()方法的参数||

## 配置历史服务
需要开启HDFS集群来存储
![1](/img/2022-08-23/4.png)


# Yarn模式
Spark 主要是计算框架，而不是资源调度框架，所以本身提供的资源调度并不是它的强项，所以还是和其他专业的资源调度框架集成会更靠谱一些。

#### 1.1修改配置文件
修改 hadoop配置文件`/opt/module/hadoop/etc/hadoop/yarn-site.xml`, 并分发
```
<!--是否启动一个线程检查每个任务正使用的物理内存量，如果任务超出分配值，则直接将其杀掉，默认
是 true -->
<property>
 <name>yarn.nodemanager.pmem-check-enabled</name>
 <value>false</value>
</property>
<!--是否启动一个线程检查每个任务正使用的虚拟内存量，如果任务超出分配值，则直接将其杀掉，默认
是 true -->
<property>
 <name>yarn.nodemanager.vmem-check-enabled</name>
 <value>false</value>
</property>
```
#### 1.2 修改 conf/spark-env.sh，添加 JAVA_HOME 和 YARN_CONF_DIR 配置
```
mv spark-env.sh.template spark-env.sh
。。。
export JAVA_HOME=/opt/module/jdk1.8.0_144
YARN_CONF_DIR=/opt/module/hadoop/etc/hadoop
```
#### 启动HDFS和YARN集群

#### 提交应用
```
bin/spark-submit \
--class org.apache.spark.examples.SparkPi \
--master yarn \
--deploy-mode cluster \
./examples/jars/spark-examples_2.12-3.0.0.jar \
10
```
# 高可用HA搭建
![1](/img/2022-08-23/5.png)
![1](/img/2022-08-23/6.png)

# 相关端口
端口号
➢ Spark 查看当前 Spark-shell 运行任务情况端口号：4040（计算） ➢ Spark Master 内部通信服务端口号：7077
➢ Standalone 模式下，Spark Master Web 端口号：8080（资源）
➢ Spark 历史服务器端口号：18080
➢ Hadoop YARN 任务运行情况查看端口号：8088