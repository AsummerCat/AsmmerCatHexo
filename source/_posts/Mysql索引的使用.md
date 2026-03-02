---
title: mysql索引的使用
date: 2018-12-05 16:07:58
tags: [mysql,数据库]
---

#  索引的使用

虽然索引大大提高了查询速度，同时却会降低更新表的速度，如对表进行INSERT、UPDATE和DELETE。因为更新表时，MySQL不仅要保存数据，还要保存一下索引文件。

建立索引会占用磁盘空间的索引文件。

<!--more-->

---



# 索引方法

Mysql目前主要有以下几种索引方法：FULLTEXT，HASH，BTREE，RTREE。

##  FULLTEXT

即为全文索引，目前只有MyISAM引擎支持。其可以在CREATE TABLE ，ALTER TABLE ，CREATE INDEX 使用，不过目前只有 CHAR、VARCHAR ，TEXT 列上可以创建全文索引。

## HASH(仅仅能满足"=","IN"和"<=>"查询，不能使用范围查询。)

由于HASH的唯一（几乎100%的唯一）及类似键值对的形式，很适合作为索引。

HASH索引可以一次定位，不需要像树形索引那样逐层查找,因此具有极高的效率。但是，这种高效是有条件的，即只在“=”和“in”条件下高效，对于范围查询、排序及组合索引仍然效率不高。

  注意:由于 Hash 索引中存放的是经过 Hash 计算之后的 Hash 值，而且Hash值的大小关系并不一定和 Hash 运算前的键值完全一样，所以数据库无法利用索引的数据来避免任何排序运算；

## BTREE (常用 二叉树)

BTREE索引就是一种将索引值按一定的算法，存入一个树形的数据结构中（二叉树），每次查询都是从树的入口root开始，依次遍历node，获取leaf。这是MySQL里默认和最常用的索引类型。



## RTREE

RTREE在MySQL很少使用，仅支持geometry数据类型，支持该类型的存储引擎只有MyISAM、BDb、InnoDb、NDb、Archive几种。

相对于BTREE，RTREE的优势在于范围查找。



---



# 索引类型



* normal：表示普通索引

* unique：表示唯一的，不允许重复的索引，如果该字段信息保证不会重复例如身份证号用作索引时，可设置为unique

* full textl: 表示 全文搜索的索引。 FULLTEXT 用于搜索很长一篇文章的时候，效果最好。用在比较短的文本，如果就一两行字的，普通的 INDEX 也可以。

总结，索引的类别由建立索引的字段内容特性来决定，通常normal最常见。
--------------------- 




---



# 基本操作



修改表结构(添加索引)

```
ALTER mytable ADD INDEX [indexName] ON (username(length)) 
```

## 创建表的时候直接指定

## 创建表的时候直接指定

```
CREATE TABLE mytable(  

ID INT NOT NULL,   

username VARCHAR(16) NOT NULL,  

INDEX [indexName] (username(length))  

);  
```

## 删除索引的语法

```
DROP INDEX [indexName] ON mytable; 
```



#  其他命令

- 查看表结构
    desc table_name;
 - 查看生成表的SQL
    show create table table_name;
 - 查看索引
    show index from  table_name;
 - 查看执行时间
    set profiling = 1;
    SQL...
    show profiles;

