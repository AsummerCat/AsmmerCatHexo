---
title: ETL工具之Sqoop使用二
date: 2022-08-01 14:17:50
tags: [ETL工具,Sqoop,大数据]
---
# ETL工具之Sqoop使用


注意一下部分`\`表示换行,可去除;

导入: 标识 RDBMS->大数据库

导出: 标识 大数据库->RDBMS


## 1. 验证sqop是否工作
```

bin/sqoop list-databases \
--connect jdbc:mysql://192.168.1.1:3306/ \
--username root \
--password 123456
```

<!--more-->

## 2. 导入数据 (import)
#### 2.1 RDBMS->HDFS (全部导入)
将某个表信息导入到HDFS
```
bin/sqoop import /
--connect jdbc:mysql://192.168.1.1:3306/company \
--username root \
--password 123456 \
--table t_order \
--target-dir /user/company \
--delete-target-dir \
--num-mappers 1 \
--fields-terminated-by "\t"
```
--target-dir            标记: 对应HDFS的目录
--delete-target-dir     标记: 目录如果存在,删除
--fields-terminated-by  标记: 指定分隔符
--num-mappers           标记: 工作线程

#### 2.2 查询导入
将查询结果导入HDFS
```
bin/sqoop import /
--connect jdbc:mysql://192.168.1.1:3306/company \
--username root \
--password 123456 \
--target-dir /user/company \
--delete-target-dir \
--num-mappers 1 \
--fields-terminated-by "\t" \
--query `select name,sex from staff where id<=1 and $CONDITIONS;`
```
无需指定表,因为使用查询语句输入
$CONDITIONS 必须要添加这个,才能执行(保证查询的顺序性)


#### 2.3 指定列导入
```
bin/sqoop import /
--connect jdbc:mysql://192.168.1.1:3306/company \
--username root \
--password 123456 \
--table t_order \
--columns id,sex \
--target-dir /user/company \
--delete-target-dir \
--num-mappers 1 \
--fields-terminated-by "\t" 

```

#### 2.4 使用sqoop关键字(where)筛选查询导入数据
```
bin/sqoop import /
--connect jdbc:mysql://192.168.1.1:3306/company \
--username root \
--password 123456 \
--table t_order \
--where "id=1" \
--target-dir /user/company \
--delete-target-dir \
--num-mappers 1 \
--fields-terminated-by "\t" 

```

#### 2.5 RDBMS导入到HIVE
```
bin/sqoop import /
--connect jdbc:mysql://192.168.1.1:3306/company \
--username root \
--password 123456 \
--table t_order \
--where "id=1" \
--num-mappers 1 \
--hive-import \
--fields-terminated-by "\t" \
--hive-overwrite \
--hive-table staff_hive
```
分为三步
1.连接mysql
2.导入到HDFS 默认位置`(/user/xxx/表名)`
3.HDFS数据迁移到hive仓库

hive-overwrite 是否覆盖(覆盖)

#### 2.6 RDBMS导入到HBASE
```
bin/sqoop import /
--connect jdbc:mysql://192.168.1.1:3306/company \
--username root \
--password 123456 \
--table t_order \
--columns "id,name,sex" \
--columns-family "info" \
--hbase-create-table \
--hbase-row-key "id" \
--hbase-table "hbase_company" \
--num-mappers 1 \
--split-by id
```
注意: `sqoop1.4.6`只支持`HBase1.0.1`之前的版本的自动创建HBase表的功能
解决方案:
手动创建表
```
create ''hbase_company,'info'
```
```
扫描HBase表:

scan 'hbase_company'
```

## 3. 导出数据 (export)

#### 3.1 将hive的结果表导出到mysql中
```
bin/sqoop export \
--connect jdbc:mysql://192.168.1.1:3306/app \
--username root \
--password 123456 \
--table t_order \
--num-mappers 1 \
--export-dir /user/hive/warehouse/app_didi.db/t_order/month=2020-04 \
--input-fields-terminated-by "\t"
```
意思是连接到mysql上的app库
--table 标识导出到mysql的t_order表
导出的目录是hive上 app_didi.db中的order表的结果
导出到mysql上

注意: 如果Mysql中如果表不存在,则不会自动创建


## 4.脚本打包
使用`opt`格式的文件打包sqoop命令,然后执行
#### 1) 创建一个.opt文件
```
mkdir opt
touch opt/job_toMYSQL.opt
```

#### 2) 编写sqoop脚本
```
export
--connect 
jdbc:mysql://192.168.1.1:3306/app 
--username
root
--password
123456
--table
t_order
--num-mappers
1
--export-dir
/user/hive/warehouse/app_didi.db/t_order/month=2020-04
input-fields-terminated-by
"\t"
```

#### 3) 执行脚本
```
bin/sqoop --options-file opt/job_toMYSQL.opt
```
