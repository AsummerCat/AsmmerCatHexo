---
title: 面试相关-mybaitis
date: 2020-02-04 11:01:57
tags: [面试相关]
---

# mybaitis

## mybatis中的#{} 和${}

```
#{} : 表示:表示一个占位符号
${}:  表示拼接sql串
```

<!--more-->

# 真题

### 当实体类中的属性名和表中的字段名不一样 ，怎么办 ?

```
1. 定义resultMap 来接收
2.修改查询语句的查询字段为实体的属性名
```

###  Dao实现的原理是什么?

```
Mapper 接口的工作原理是JDK动态代理，Mybatis运行时会使用JDK动态代理为Mapper接口生成代理对象proxy，代理对象会拦截接口方法，转而执行MapperStatement所代表的sql，然后将sql执行结果返回。
```

### 传入参数如何处理?

```
@param
或者传入Map
```

### mybatis和mybaitis-plus的区别?

```
1.plus 内置分页插件
2.plus 有通用的service 可以简单配置实现单表查询 类似jpa
```

