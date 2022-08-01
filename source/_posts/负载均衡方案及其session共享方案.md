---
title: 负载均衡方案及其session共享方案
date: 2019-04-16 09:31:53
tags: [java]
---

# 负载均衡方案及其session共享方案

## 负载均衡方案介绍

### F5 硬件加速



### HAProxy

工作第四/器层

session 保持,并发处理极佳  ,负载均衡算法多



### LVS 

工作第四层

工作稳定,应用范围广,配置简单 

<font color="red">不能做动静分离</font>

<!--more-->

### Nginx (常用)

安装配置简单 

占有内存少

并发处理能力强 3w+ 优化下10w+

功能强大,动静分离,反向代理,Lua脚本支持

<font color="red">工作于第七层(支持应用少)不能保持session</font>

请自己选择seesion方案  seesion 同步 复制 粘性



####  Apache





## SESSION共享方案

### 大型分布式环境首选,可扩展性强,健壮

seesion 共享    

解决方案: 整合redis 实现seesion 或者zk的session共享



### 较小分布式环境首选,无入侵,健壮无单点故障问题

session 复制    7台以下服务器

解决方案: 使用tomcat自带的seesion复制方案 配置简单



### 配置简单 ,无入侵,存在单点故障问题

负载均衡

seesion ip_hash 

使用nginx的 这样可以把同一ip的请求 绑定到一台服务器上 但是一旦挂机 就挂了

