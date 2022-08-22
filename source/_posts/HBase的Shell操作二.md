---
title: HBase的Shell操作二
date: 2022-08-22 11:30:49
tags: [大数据,HBase]
---
# HBase的Shell操作

## 进入HBase客户端命令
```
bin/hbase shell
```
#### 查看帮助命令
```
help
```
<!--more-->
## namespace相关命令
#### 查看命名空间
```
list_namespace
或者
list_namespace 'abc.*'
```
#### 创建命名空间
```
create_namespace 'ns1'
```
## DDL命令 表格table相关

#### 查看表格
```
list
```
#### 查看表格详细信息
```
describe '表格名称'
```
#### 创建表格
```
NAME 列族名称 , VERSIONS 当前版本号
create '命名空间:表格名称', {NAME => '列族名称',VERSIONS => 5}

或
默认命名空间里,创建表格t1 + 对应f1,f2列族 并且版本号默认为1
create 't1', 'f1','f2'
```

#### 修改列族和删除列族
```
(1) 增加列族和修改信息都使用覆盖的方法

alter '表格名称', {NAME => '列族名称',VERSION => 3}

(2) 删除列族信息使用特殊的语法

alter '表格名称', NAME => 'f1', METHOD =>'delete'

alter '表格名称', 'delete' => 'f1'
```

#### 删除表格
```
shell删除表格,需要先将表格状态设置为不可用

disable '表格名称' 
drop '表格名称'
```

#### 写入数据
如果重复写入相同的`rowKey`,相同列的数据,会写入多个版本覆盖
```
语法:
put '命名空间:表格名称','行号','列族:列名','具体值'

例子:
put '命名空间:表格名称','1001','info:name','zhangsan'
put '命名空间:表格名称','1001','info:name','lishi'
put '命名空间:表格名称','1001','info:age','18'
```

#### 获取数据
```
语法:
get '命名空间:表格名称','行号'

例子:
get 'bigdata:student','1001'

查询指定字段
get 'bigdata:student','1001',{COLUMN => 'info:name'}
```

#### 删除数据
```
delete 表示删除一个版本的数据，即为 1 个 cell，不填写版本默认删除最新的一个版本。
delete '命名空间:表格名称','行号','列族:列名'

delete 'bigdata:student','1001','info:name'

deleteall 表示删除所有版本的数据，即为当前行当前列的多个 cell。（执行命令会标记
数据为要删除，不会直接将数据彻底删除，删除数据只在特定时期清理磁盘时进行）

deleteall 'bigdata:student','1001','info:name'
```