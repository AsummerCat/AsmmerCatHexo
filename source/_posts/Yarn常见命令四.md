---
title: Yarn常见命令四
date: 2022-08-01 16:14:28
tags: [大数据,hadoop,yarn]
---
# Yarn常见命令
除了可以在网页上 `ip:8088`页面查看外,还可以通过命令行的方式操作



## yarn application 查看任务

#### 1.列出所有Application
```
yarn application -list
```
<!--more-->
#### 2.根据Application状态过滤
```
yarn application -list -appStates FINISHED 

所有状态(
ALL, NEW ,NEW_SAVING , SUMBITTED ,ACCEPTED,
RUNNING ,FINISHED , FAILED , KILLED
)
```

#### 3.kill掉Application
```
yarn application -kill  application_id
```


## yarn logs 查看日志

#### 1.查看Application日志 (生产用的多)
```
yarn logs -applicationId 对应id号
```

#### 2.查看Container日志
```
yarn logs -applicationId 对应id号 -containerId 容器号
```



## yarn applicationattempt 查看尝试运行的任务
#### 1.列出所有Application尝试的列表
```
yarn applicationattempt -list applicationId

能看到对应的容器
```
#### 2.打印 Applicationattempt 状态
```
yarn applicationattempt -status applicationId
```

## yarn container 查看容器
#### 1.列出所有Container
```
yarn container -list applicationattemptId

```

#### 2.打印container状态
```
yarn container -status containerId
```
<font color='red'>注:只有在任务跑的途中才能看到container的状态</font>

## yarn node 查看节点状态
#### 1.列出所有节点
```
yarn node -list -all
```

## yarn rmadmin 更新配置
#### 1.加载队列配置
```
yarn rmaadmin -refreshQueues
```

## yarn queue 查看队列
#### 1.打印队列信息
```
yarn queue -status queueName队列名称
```
