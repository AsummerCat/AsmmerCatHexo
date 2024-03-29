---
title: ETL工具之Sqoop常用命令三
date: 2022-08-01 14:18:21
tags: [ETL工具,Sqoop,大数据]
---
# ETL工具之Sqoop常用命令

sql to hadoop
https://sqoop.apache.org/docs/1.4.7/SqoopUserGuide.html
##  命令&参数详解
刚才列举了一些Sqoop的常用命令，对于不同的命令，有不同的参数，让我们来一一列举说明。
首先来我们来介绍一下公用的参数，所谓公用参数，就是大多数命令都支持的参数。

<!--more-->
#### 5.2.1 公用参数：数据库连接

| 序号 | 参 数| 说明 |
| --- | --- | --- |
|1|–connect|连接关系型数据库的URL|
|2|–connection-manager|指定要使用的连接管理类|
|3|–driverHadoop|根目录|
|4|–help|打印帮助信息|
|5|–password|连接数据库的密码|
|6|–username|连接数据库的用户名|
|7|–verbose|在控制台打印出详细信息|

#### 5.2.2 公用参数：import

|序号| 参数                             |说明|
| --- |--------------------------------| --- |
|1| –enclosed-by <char>            |给字段值前加上指定的字符|
|2| –escaped-by <char>             |对字段中的双引号加转义符|
|3| –fields-terminated-by <char>   |设定每个字段是以什么符号作为结束，默认为逗号|
|4| –lines-terminated-by <char>    |设定每行记录之间的分隔符，默认是\n|
|5| –mysql-delimitersMysql         |默认的分隔符设置，字段之间以逗号分隔，行之间以\n分隔，默认转义符是\，字段值以单引号包裹。|
|6| –optionally-enclosed-by <char> |给带有双引号或单引号的字段值前后加上指定字符。|
#### 5.2.3 公用参数：export
|序号| 参数                                   |说明|
| --- |--------------------------------------| --- |
|1| –input-enclosed-by <char>            |对字段值前后加上指定字符|
|2| –input-escaped-by <char>             |对含有转移符的字段做转义处理|
|3| –input-fields-terminated-by <char>   |字段之间的分隔符|
|4| –input-lines-terminated-by <char>    |行之间的分隔符|
|5| –input-optionally-enclosed-by <char> |给带有双引号或单引号的字段前后加上指定字符|

#### 5.2.4 公用参数：hive
|序号| 参数                             |说明|
| --- |--------------------------------| --- |
|1| –hive-delims-replacement <arg> |用自定义的字符串替换掉数据中的\r\n和\013 \010等字符|
|2| –hive-drop-import-delims       |在导入数据到hive时，去掉数据中的\r\n\013\010这样的字符|
|3| –map-column-hive <arg>         |生成hive表时，可以更改生成字段的数据类型|
|4| –hive-partition-key            |创建分区，后面直接跟分区名，分区字段的默认类型为string|
|5| –hive-partition-value <v>      |导入数据时，指定某个分区的值|
|6| –hive-home <dir>               |hive的安装目录，可以通过该参数覆盖之前默认配置的目录|
|7| –hive-import                   |将数据从关系数据库中导入到hive表中|
|8| –hive-overwrite                |覆盖掉在hive表中已经存在的数据|
|9| –create-hive-table             |默认是false，即，如果目标表已经存在了，那么创建任务失败。|
|10| –hive-table                    |后面接要创建的hive表,默认使用MySQL的表名|
|11| –table                         |指定关系数据库的表名|
