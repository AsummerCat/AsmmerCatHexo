---
title: spi扩展机制
date: 2020-04-08 21:29:57
tags: [java,Spring]
---

# SPI扩展机制

# 配置

```
代码中加载
ServiceLoader<接口名称> docs= ServiceLoader.load(接口.class)

docs.xxxxxx()

配置文件中修改
resourcess->META-INF->sercices
创建接口全类名 
里面写入需要实现的实现类全类名

这样load出来的就是实现类了
```