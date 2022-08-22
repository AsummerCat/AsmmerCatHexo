---
title: HBase整合Phoenix七
date: 2022-08-22 11:34:08
tags: [大数据,HBase,Phoenix]
---
# HBase整合Phoenix
`这样可以用sql的写法来处理HBase的结构`

Phoenix 是 HBase 的开源 SQL 皮肤。可以使用标准 JDBC API 代替 HBase 客户端 API
来创建表，插入数据和查询 HBase 数据。

```
在 Client 和 HBase 之间放一个 Phoenix 中间层不会减慢速度，因为用
户编写的数据处理代码和 Phoenix 编写的没有区别，不仅如此
Phoenix 对于用户输入的 SQL 同样会有大量的优化手段（就像 hive 自带 sql 优化器一样）
```
<!--more-->
## 1.安装
官网地址
http://phoenix.apache.org/
略
#### 1.1 复制 server 包并拷贝到各个节点的 hbase/lib
```
cp phoenix-server-hbase-2.4-5.1.2.jar /opt/module/hbase/lib/
```
#### 1.2 配置环境变量
```
#phoenix
export PHOENIX_HOME=/opt/module/phoenix
export PHOENIX_CLASSPATH=$PHOENIX_HOME
export PATH=$PATH:$PHOENIX_HOME/bin
```
#### 1.3 重启 HBase
```
 stop-hbase.sh
 start-hbase.sh
```
## 2.Phoenix Shell 操作
#### 2.1 显示所有表
```
!table 或 !tables
```
#### 2.2 创建表
直接指定单个列作为 RowKey
```
CREATE TABLE IF NOT EXISTS student(
id VARCHAR primary key,
name VARCHAR,
age BIGINT,
addr VARCHAR);
```
在 phoenix 中，表名等会自动转换为大写，若要小写，使用双引号，如"us_population"。
指定多个列的联合作为 RowKey
```
CREATE TABLE IF NOT EXISTS student1 (
id VARCHAR NOT NULL,
name VARCHAR NOT NULL,
age BIGINT,
addr VARCHAR
CONSTRAINT my_pk PRIMARY KEY (id, name));
```
在创建表语句末尾添加`COLUMN_ENCODED_BYTES = 0` 可在HBase中显示完整字符

#### 2.3 插入数据 更新数据 二合一
```
upsert into student values('1001','zhangsan', 10, 'beijing');
```
#### 2.4 查询记录
```
select * from student;
select * from student where id='1001';
```
#### 2.5 删除记录
```
delete from student where id='1001';
```
#### 2.6 删除表
```
drop table student;
```
#### 2.7 退出命令行
```
!quit
```

# 3.Phoenix JDBC 操作

#### 3.1 导入pom依赖
```
<dependencies>
 <dependency>
 <groupId>org.apache.phoenix</groupId>
 <artifactId>phoenix-client-hbase-2.4</artifactId>
 <version>5.1.2</version>
 </dependency>
</dependencies>
```
#### 3.2 编写代码
```
import java.sql.*;
import java.util.Properties;
public class PhoenixClient {
 public static void main(String[] args) throws SQLException {
 // 标准的 JDBC 代码
 // 1.添加链接
  String url = "jdbc:phoenix:hadoop102,hadoop103,hadoop104:2181";
 // 2. 创建配置
 // 没有需要添加的必要配置 因为 Phoenix 没有账号密码
 Properties properties = new Properties();
 // 3. 获取连接
 Connection connection = DriverManager.getConnection(url, properties);
 // 5.编译 SQL 语句
 PreparedStatement preparedStatement = connection.prepareStatement("select * from student");
 // 6.执行语句
 ResultSet resultSet = preparedStatement.executeQuery();
 // 7.输出结果
 while (resultSet.next()){
 System.out.println(resultSet.getString(1) + ":" + resultSet.getString(2) + ":" + resultSet.getString(3));
 }
 // 8.关闭资源
 connection.close();
 // 由于 Phoenix 框架内部需要获取一个 HBase 连接,所以会延迟关闭
 // 不影响后续的代码执行
 System.out.println("hello");
 } 
}
```