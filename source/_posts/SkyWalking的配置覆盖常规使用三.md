---
title: SkyWalking的配置覆盖常规使用三
date: 2022-02-09 22:19:23
tags: [链路跟踪,SkyWalking]
---

# SkyWalking的配置覆盖

## SkyWalking的常规使用
以配置覆盖的形式输出 这样就不用每次需要创建一个新的探针了

<!--more-->

# 支持的形式

## 1.第一种->系统配置(System properties)
使用`SkyWalking.` +配置文件中的配置名作为系统配置项来进行覆盖  
例如:  
通过 如下进行`agent.service_name`的覆盖
```
jvm参数上添加 这个参数可以有效的隔离

-Dskywalking.agent.service_name=项目名称
```


## 2. 第二种->探针配置(Agent options)
jvm启动参数上添加
```
标准格式:

-javaagent:/path/xxx-agent.jar=[key]=[value],[key]=[value]
```
例如:
```
-javaagent:/path/xxx-agent.jar=agent.service_name=项目名称
```
## 3.系统环境变量
```
由于配置项是 
agent.service_name
默认的是${SW_AGENT_NAME:your_ApplicationName}

所以可以在环境变量中设置
SW_AGENT_NAME 来指定服务名

```
### 注意:
如果有特殊字符
```
分隔符 ( , 或者 =)
就必须用引号包裹起来
`xxx`
```

# 配置覆盖优先级
```
探针配置>系统配置>系统环境变量配置>配置文件中的值
```
