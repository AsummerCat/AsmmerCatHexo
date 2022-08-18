---
title: scala集合的常用函数
date: 2022-08-18 17:45:44
tags: [大数据,scala]
---
# scala集合的常用函数

## 基本属性和常用操作
1) 获取集合长度
```
list.length
```
2) 获取集合大小
```
set.size
```
<!--more-->
3) 遍历循环
```
for (elem -> list)
  println(elem)
  
set.foreach(println)  
```
4) 迭代器
```
for (elem -> list.iterator)
  println(elem)
```
5) 生成字符串
```
println(list)
println(set)

println(list.mkString("-"))
```
6) 是否包含
```
println(list.contains("xx"))
println(set.contains("xx"))
```

## 其他操作
1) 获取集合的头
```
list.head
```
2) 获取集合的 除了第一个之外的数据
```
list.tail
```
3) 获取最后一个数据
```
list.last
```
4) 集合初始数据 (获取除了最后一个元素)
```
list.init
```
5) 反转 reverse
```
list.reverse
```
6) 取前(后)n个元素 take
```
list.take(3)

//取右边开始计算的n个元素
list.takeRight(3)

```
7) 去掉前(后)n个元素 drop
```
list.drop(3)

//去掉右边开始计算的n个元素
list.dropRight(3)
```
8) 并集 union
```
list.union(list2)

set.union(set1)
```
9) 交集 intersect
```
list.intersect(list2)
```
10) 差集 diff
```
list.diff(list2)
```
11) 拉链->对应位置 形成一个二元数组
```
list.zip(list2)
```

# 集合计算简单函数
1) 求和
```
list.sum
```
2) 求乘积
```
list.product
```
3) 最大值
```
list.max

list.maxBy((tuple: (String,Int))=>tuple._2)
等价
list.maxBy(_._2)
```
4) 最小值
```
list.min
```
5) 排序
```
//从大到小逆序排序
list.sorted.reverse
```


