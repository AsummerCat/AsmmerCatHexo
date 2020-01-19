---
title: jmeter登录授权配置
date: 2020-01-19 16:53:21
tags: [java,压测]
---

# jmeter登录授权配置

jmeter, Apache下的测试工具, 常用来进行压测, 项目中, 接口通常都需要进行登录才能被调用, 直接调用将提示"登录失效", 下面介绍如何在jmeter中配置参数实现登录

或者

手动登录一次，使用Charles或者其他抓包工具查看数据包，可以看到cookie，拿到后
放到postman对应的headers里，就可以进行正常的接口测试了。

## 整体思路

1. 配置header
2. 通过浏览器调试工具(F12)抓取Cookie
3. 配置cookie
4. 登录
5. 测试

## 实现步骤

<!--more-->

### 配置header

![添加cookie](/img/2020-01-15/12.png)