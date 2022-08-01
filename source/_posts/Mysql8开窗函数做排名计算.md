---
title: Mysql8开窗函数做排名计算
date: 2022-08-01 13:55:07
tags: [mysql,hive]
---
# Mysql8开窗函数
这里主要是计算排名的

比如:
```
row_number:不管排名是否有相同的,都按照顺序1,2,3...n;

1
2
3
4

rank: 排名相同的名次一样,同一排名有几个,后面排名就会跳过几次
1
2
2
4

dense_rank: 排名相同的名次一样,且后面名次不跳跃;
1
2
2
3


```
<!--more-->

## 例子:
根据`deptid` 分组,再根据salry进行排序;
```
select 
   empid,
   deptid,
   salary,
   row_number() over (PARTITION BY deptid ORDER BY salary desc ) as row_number1 
   from employee;
 
```