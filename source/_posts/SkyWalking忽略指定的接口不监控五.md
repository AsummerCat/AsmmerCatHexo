---
title: SkyWalking忽略指定的接口不监控五
date: 2022-02-09 22:22:40
tags: [链路跟踪,SkyWalking]
---

# SkyWalking忽略指定的接口不监控

## 部署方式
进入安装SkyWalking的目录
将`/agent/optional-plugins/apm-trace-igonre-plugin-6.4.0.jar`插件拷贝到`plugins`目录中

<!--more-->
然后
```
cd bin

使用 start.sh 一次性启动两个服务

```
即可

## 使用方式

jvm参数即可运行

```
jvm启动配置项中添加

-Dskywalking.agent.trace.ignore_path=/xxx接口路径

支持的表达式:
/path/* , /path/** ,/path/?
```
