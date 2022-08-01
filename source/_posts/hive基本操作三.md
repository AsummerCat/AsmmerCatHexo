---
title: hive基本操作三
date: 2022-08-01 16:05:01
tags: [大数据,hadoop,hive]
---
# Hive基本操作

## 开发工具
vscode里有可视化操作 ->插件->hive sql插件

datagrip也行

注意: hive的脚本后缀是 -> *.hql

## 数据库操作:

#### 1.创建数据库-默认方式
```
create database if not exists myhive;

1. if not exists : 如果数据存在则不创建,不存在则创建

2. hiave的数据库默认存放在'/user/hiave/warehouse'目录 (这个目录是在hdfs上的)
```

#### 2.创建数据库-指定存储路径

```
create database db location '/myhive2';

1.说明:
 location : 用来指定数据库的存放目录

```

<!--more-->

#### 3.删除数据库
删除一个空数据库,如果数据库下面有数据表,那么就会报错
```
drop database myhive;

```
强制删除数据库,包含数据库下面的表一起删除
```
drop database myhive2 cascade;
```

#### 4. 查看某个数据库详细信息
```
desc database myhive;
```

# 建表语句
跟mysql差不多

#### 1.创建数据表 (分为内部表和外部表)

##### 内部表
未被`external`修饰的是内部表(managed table), 内部表又称管理表. 内部表不适合用于共享数据;
```
create table stu( id int ,name string);
```
创建表之后,hivbe会在对应数据库文件夹下创建对应的表目录;
`show tables`

ps:
1.hive内部表在信息drop操作时,其表中的数据以及标的源数据信息均会被删除
2.<font color='red'>内部表一般可以用来做中间表或者临时表</font>

##### 外部表(外部表数据可用于共享)   external
需要建立关键字`external`;
创建表时,使用`external`关键字来修饰则为外部表,外部表数据可用于共享;
```
create external table stu( id int ,name string)
row format delimited fields terminated by '\t' 
location '/hive_table/student';

说明:
location: 表文件的存储路径
```
创建表之后,hive会在对应数据库文件夹下创建对应的表目录;

ps:
1.外部表在进行drop操作的时候,仅会删除元数据,而不删除HDFS上的文件(这样数据源文件仍然存在,仅做逻辑删除)
2.外部表一般用于数据共享表,比较安全;

#### 2.查看表结构

 ```
 desc stu ; 查看结构本信息
 
 desc formatted stu; 查看表结构详细信息
 ```

#### 3.删除表

```
drop table tu;
```

# 表操作语句
#### 1.插入表数据
```
insert into stu values(1,'xxxx');
```
该方式 会在hdfs中插入一个文件,不推荐使用;一条记录一个小文件  
推荐使用load命令

#### 2.load加载数据
Load命令用于将外部加载到Hive表;
语法:
```
LOAD DATA local inpath 'xxxfilepath' into table stu2;


LOAD DATA local inpath 'xxxfilepath' overwrite into table stu2;

说明:
   1.local,  是否本地文件加载,否则是从HDFS加载
   2.overwrite 表数据是否需要覆盖已有数据



```

本地加载,建表手动指定分隔符
```
create table if not exists stu2(id int,name string)
row format delimited fields terminated by '\t' ;

# 向表加载数据
load data local inpath '/export/data/hivedatas/stu.txt' into table stu2;
```
文件会直接存储在HDFS上


# 分区表操作
可以简单理解为: 根据某个字段进行分类存储,
跟orcle的分区表很像;

分区表实际就是对应hdfs系统上的独立文件夹;

#### 创建一级分区表
```
create table score (sid string,cid string)
partitioned by (month string) 
row format delimited fields terminated by '/t';

根据月度分区  month:指定分区字段
```

#### 数据加载
```
local data local inpath '/xxx.txt' into table score
  partition (month='202006');
```

#### 多级分区表

```
create table score (sid string,cid string)
partitioned by (year string,month string,day string) 
row format delimited fields terminated by '/t';

根据年月日分区  year,month,day:指定分区字段
```
数据加载:
```
local data local inpath '/xxx.txt' into table score
  partition (year='2020',month='06',day='01');
```
加载数据之后,多级分区表会创建多级分区信息;

#### 查看分区
```
show partition 表名;
```

#### 添加分区
```
添加一个分区
alter table 表名 add partition(month='202008');

添加多个分区
alter table 表名 add partition(month='202009') partition(month='202010');
```

#### 删除分区
```
alter table 表名 drop partition(month='202008');
```



# 查询操作

跟mysql 类似

