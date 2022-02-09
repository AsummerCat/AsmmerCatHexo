---
title: SkyWalking监控告警六
date: 2022-02-09 22:23:12
tags: [链路跟踪,SkyWalking]
---

# SkyWalking监控告警
可以根据`服务响应时间`,`服务响应时间百分比`等进行告警

## 默认告警规则
位于`skyWalking`安装目录下的`config`文件夹下`alarm-settings.yml`文件中:

```
rules: 
 xxxx:
 
 xxxx_rule: 每一条都代表一个告警规则
 
 
webhooks: 
//告警产生后 的回调地址
//然后我们可以自定义监控 比如发送短信 之类的
```
