---
title: Flink启动方式二
date: 2022-09-19 16:55:00
tags: [大数据,Flink]
---
# Flink 启动方式

## 本地启动

最简单的启动方式，其实是不搭建集群，直接本地启动。本地部署非常简单，直接解压安装包就可以使用，不用进行任何配置；一般用来做一些简单的测试。

启动

    $ cd flink-1.13.0/
    $ bin/start-cluster.sh

关闭集群

    bin/stop-cluster.sh

访问 Web UI
启动成功后，访问` http://hadoop102:8081`，可以对 flink 集群和任务进行监控管理

<!--more-->

## 向集群提交作业

打 包 完 成 后 ， 在 target 目 录 下 即 可 找 到 所 需 jar 包 ， jar 包 会 有 两 个 ，FlinkTutorial-1.0-SNAPSHOT.jar 和 FlinkTutorial-1.0-SNAPSHOT-jar-with-dependencies.jar，因为集群中已经具备任务运行所需的所有依赖，所以建议使用 FlinkTutorial-1.0-SNAPSHOT.jar。 2.

        <build>
            <plugins>
                <plugin>
                    <groupId>org.apache.maven.plugins</groupId>
                    <artifactId>maven-assembly-plugin</artifactId>
                    <version>3.0.0</version>
                    <configuration>
                        <descriptorRefs>
                            <descriptorRef>jar-with-dependencies</descriptorRef>
                        </descriptorRefs>
                    </configuration>
                    <executions>
                        <execution>
                            <id>make-assembly</id>
                            <phase>package</phase>
                            <goals>
                                <goal>single</goal>
                            </goals>
                        </execution>
                    </executions>
                </plugin>
            </plugins>
        </build>

### web页面提交job

（1）任务打包完成后，我们打开 Flink 的 WEB UI 页面，在右侧导航栏点击“Submit New Job”，然后点击按钮“+ Add New”，选择要上传运行的 jar 包

(2) 点击该 jar 包，出现任务配置页面，进行相应配置。主要配置程序入口主类的全类名，任务运行的并行度，任务运行所需的配置参数和保存点路径等，如图 3-6 所示，配置完成后，即可点击按钮“Submit”，将任务提交到集群运行。

(3)任务提交成功之后，可点击左侧导航栏的“Running Jobs”查看程序运行列表情况

（4）点击该任务，可以查看任务运行的具体情况，也可以通过点击“Cancel Job”结束任务运行

### 命令行提交作业

进入到 Flink 的安装路径下，在命令行使用 flink run 命令提交作业

    bin/flink run -m hadoop102:8081 -c com.xx.wc.StreamWordCount ./FlinkTutorial-1.0-SNAPSHOT.jar

    这里的参数 –m 指定了提交到的 JobManager，-c 指定了入口类

