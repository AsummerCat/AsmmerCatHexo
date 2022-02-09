---
title: SkyWalking的安装一
date: 2022-02-09 22:17:50
tags: [链路跟踪,SkyWalking]
---

# SkyWalking的安装

# 跟CAT的区别
SkyWalking : 非侵入式
CAT: 侵入式 可以进行更详细的监控

## 需要安装2个
```
1.安装Backend后端服务
2.安装UI
```

## 安装步骤
1. 解压压缩包  

<!--more-->

2. 修改数据源
```
cd apache-skyWalking-apm-bin
vi config application.yml
```
修改里面的数据源配置
```
storage里的配置 修改为mysql或者es即可
默认使用h2数据库
```
3. 修改ui的配置
```
cd webapp
修改 webapp.yml

server:
   port: 修改ui的端口
```

## 启动SkyWalking
```
cd bin

oapService.sh 启动后端服务

webappService.sh 启动ui服务

使用 start.sh 一次性启动两个服务
```

## 修改探针 使其对应应用
```
cd agent/config

vi agent.config

修改:
agent.service_name={SW_AGENT_NAME:your_applicationame}

为你的项目名称:
agent.service_name={SW_AGENT_NAME:test1}
```
