---
title: 一个Object对象占用多少字节
date: 2020-06-03 21:34:05
tags: [java]
---

# 一个Object对象占用多少字节

```
Object o=new Object();   一般情况16字节
  
  64位 8个字节
```

## jvm参数

```
-XX:+UseCompressedClassPointers   压缩类的指针
-XX:+UseCompressedOops   压缩普通对象指针
```

<!--more-->

```
结构:64位系统
8个字节的 mark word  (消息头)
8个字节的   如果开启了类的指针压缩:=>4个字节  -XX:+UseCompressedClassPointers
成员变量: 如果开启了压缩普通对象指针:=>4个字节  -XX:+UseCompressedOops 

```

