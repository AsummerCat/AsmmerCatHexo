---
title: StringJoiner拼接字符串
date: 2021-10-08 23:59:00
tags: [java]
---

# StringJoiner拼接字符串

## 字符串拼接
```
       //字符串拼接 ,分割 并且配置开头和结尾 注意: 开头结尾存在则 如何没有内容的话 还是会开头结尾拼接输出
        StringJoiner stringJoiner = new StringJoiner(",", "[", "]");
        stringJoiner.add("xiao");
        stringJoiner.add("zhi");
        System.out.println(stringJoiner.toString());
```
<!--more-->
## SQL的in拼接

```
StringJoiner在处理sql拼接上面，也非常方便，如拼接 sql 的in条件的时候：

StringJoiner joiner3 = new StringJoiner("','", "'", "'");
joiner3.add("1").add("2");
//输出 ： '1','2'
```

## 直接拼接
```
 StringJoiner stringJoiner = new StringJoiner(",");
        stringJoiner.add("xiao");
        stringJoiner.add("zhi");
        System.out.println(stringJoiner.toString());
    
输出:xiao,zhi        
```
