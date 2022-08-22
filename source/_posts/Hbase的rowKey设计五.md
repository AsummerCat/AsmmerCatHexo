---
title: Hbase的rowKey设计五
date: 2022-08-22 11:33:00
tags: [大数据,HBase]
---
# HBase的rowKey设计
一条数据的唯一标识就是`rowKey`,那么这条数据存储与哪个分区,取决于`rowKey`处于哪一个预分区的区间内;

设计`rowKey`的主要目的:就是让数据均匀的分布于所有`region`中,在一定程度上防止数据倾斜;

## rowKey设计 = 表格的设计

#### 常规设计 (TSDB)
多行存储 -> 存储字段变化记录

`rowKey`的设计方案:
1. 生成随机数,hash,散列值  (打散数据)
2. 时间戳反转
3. 字符串拼接

<!--more-->
<font color='red'>ps:一种设计格式只能完美的满足一个需求
一个设计可以兼容多个需求;
设计方向: 可以穷举的写在前面,减少范围
</font>

<font color='red'>案例一:</font>
![image](/img/2022-08-22/11.png)

<font color='red'>案例二:</font>
![image](/img/2022-08-22/12.png)

<font color='red'>合并案例一二设计:</font>
![image](/img/2022-08-22/13.png)


#### 添加预分区(预分区的优化)
![image](/img/2022-08-22/14.png)
![image](/img/2022-08-22/15.png)
