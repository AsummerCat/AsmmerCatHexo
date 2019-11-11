---
title: 利用B表数据批量更新A表数据
date: 2019-11-11 13:37:07
tags: [数据库]
---

# 根据B表字段 更新A表对应的字段

比如:

```
A表                             B表
id name age sex               id age page
1  小明       男               1  19  2
2  小抽       女               2  20  2

更新后的效果:
A表                          
id name age sex            
1  小明  19  男              
2  小抽  20  女               
```

<!--more-->

## mysql

```
UPDATE TableA a,TableB b SET a.value = b.value WHERE a.key = b.key
```



## orcle

```
update TEST_1 A set A.CREATE_TIME= ( select CREATE_TIME from TEST_2 b where B.id = A.id)
```



## sqlService

```java
update a set a.value=b.value from tableA a, tableB b where a.key=b.key
```

