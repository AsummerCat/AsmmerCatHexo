---
title: SpringBoot添加自定义配置的属性@value
date: 2020-03-21 21:53:45
tags: [java,SpringBoot]
---

# SpringBoot添加自定义配置的属性@value

### 创建自定义配置 test.properties

```
test.haha=111
```

<!--more-->

### 在需要的类上 或者启动类上加入

```
@PropertySource({"classpath:test.properties"}) //标识读取该配置
```

### 在需要使用属性的地方加入@value

```
 @Value("${test.haha}")
    private int haha;
```

