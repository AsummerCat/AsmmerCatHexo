---
title: HBase的API操作三
date: 2022-08-22 11:31:44
tags: [大数据,HBase]
---
# HBase的API操作

## 导入pom
```
        <dependencies>
            <dependency>
                <groupId>org.apache.hbase</groupId>
                <artifactId>hbase-server</artifactId>
                <version>2.4.11</version>
                <exclusions>
                    <exclusion>
                       <groupId>org.glassfish</groupId>
                       <artifactId>javax.el</artifactId>
                    </exclusion>
                </exclusions>
            </dependency>
            <dependency>
                <groupId>org.glassfish</groupId>
                <artifactId>javax.el</artifactId>
                <version>3.0.1-b06</version>
            </dependency>
        </dependencies>
```
<!--more-->

## 操作

#### 单线程获取链接
```
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.hbase.client.AsyncConnection;
import org.apache.hadoop.hbase.client.Connection;
import org.apache.hadoop.hbase.client.ConnectionFactory;

import java.io.IOException;
import java.util.concurrent.CompletableFuture;

public class HBaseConnect {
    public static void main(String[] args) throws IOException {
        // 1. 创建配置对象
        Configuration conf = new Configuration();
        // 2. 添加配置参数
        conf.set("hbase.zookeeper.quorum", "hadoop102,hadoop103,hadoop104");
        // 3. 创建 hbase 的连接
        // 默认使用同步连接
        Connection connection = ConnectionFactory.createConnection(conf);
        // 可以使用异步连接
        // 主要影响后续的 DML 操作
        CompletableFuture<AsyncConnection> asyncConnection = ConnectionFactory.createAsyncConnection(conf);
        // 4. 使用连接
        System.out.println(connection);
        // 5. 关闭连接
        connection.close();
    }
}
```

#### 多线程创建连接 (推荐) 单实例 静态
```
package com.longshine.fuzhou.operation_center.aspect;

import org.apache.hadoop.hbase.client.Connection;
import org.apache.hadoop.hbase.client.ConnectionFactory;

import java.io.IOException;

public class HBaseConnect {
    // 设置静态属性 hbase 连接
    public static Connection connection = null;

    static {
        // 创建 hbase 的连接
        try {
            // 使用配置文件的方法
            connection = ConnectionFactory.createConnection();
        } catch (IOException e) {
            System.out.println("连接获取失败");
            e.printStackTrace();
        }
    }

    /**
     * 连接关闭方法,用于进程关闭时调用
     *
     * @throws IOException
     */
    public static void closeConnection() throws IOException {
        if (connection != null) {
            connection.close();
        }
    }
}
```
#### 在 resources 文件夹中创建配置文件 hbase-site.xml，添加以下内容
```
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
 <property>
 <name>hbase.zookeeper.quorum</name>
 <value>hadoop102,hadoop103,hadoop104</value>
 </property>
</configuration>
```

# DDL
创建 HBaseDDL 类，添加静态方法即可作为工具类
```
public class HBaseDDL {
 // 添加静态属性 connection 指向单例连接
 public static Connection connection = HBaseConnect.connection;
}
```

#### 创建命名空间
```
/**
 * 创建命名空间
 *
 * @param namespace 命名空间名称
 */
public static void createNamespace(String namespace)throws IOException{
        // 1. 获取 admin
        // 此处的异常先不要抛出 等待方法写完 再统一进行处理
        // admin 的连接是轻量级的 不是线程安全的 不推荐池化或者缓存这个连接
        Admin admin=connection.getAdmin();
// 2. 调用方法创建命名空间
// 代码相对 shell 更加底层 所以 shell 能够实现的功能 代码一定能实现
// 所以需要填写完整的命名空间描述
        // 2.1 创建命令空间描述建造者 => 设计师
        NamespaceDescriptor.Builder builder=NamespaceDescriptor.create(namespace);
        // 2.2 给命令空间添加需求
        builder.addConfiguration("user","atguigu");
        // 2.3 使用 builder 构造出对应的添加完参数的对象 完成创建
        // 创建命名空间出现的问题 都属于本方法自身的问题 不应该抛出
        try{
        admin.createNamespace(builder.build());
        }catch(IOException e){
        System.out.println("命令空间已经存在");
        e.printStackTrace();
        }
        // 3. 关闭 admin
        admin.close();
        }
```

#### 判断表格是否存在
```
/**
 * 判断表格是否存在
 *
 * @param namespace 命名空间名称
 * @param tableName 表格名称
 * @return ture 表示存在
 */
public static boolean isTableExists(String namespace,String
        tableName)throws IOException{
        // 1. 获取 admin
        Admin admin=connection.getAdmin();
        // 2. 使用方法判断表格是否存在
        boolean b=false;
        try{
        b=admin.tableExists(TableName.valueOf(namespace,tableName));
        }catch(IOException e){
        e.printStackTrace();
        }
        // 3. 关闭 admin
        admin.close();
        // 3. 返回结果
        return b;
        // 后面的代码不能生效
        }
```

#### 创建表
```
/**
 * 创建表格
 *
 * @param namespace 命名空间名称
 * @param tableName 表格名称
 * @param columnFamilies 列族名称 可以有多个
 */
public static void createTable(String namespace,String tableName,String...columnFamilies)throws IOException{
        // 判断是否有至少一个列族
        if(columnFamilies.length==0){
        System.out.println("创建表格至少有一个列族");
        return;
        }
        // 判断表格是否存在
        if(isTableExists(namespace,tableName)){
        System.out.println("表格已经存在");
        return;
        }
        // 1.获取 admin
        Admin admin=connection.getAdmin();
        // 2. 调用方法创建表格
        // 2.1 创建表格描述的建造者
        TableDescriptorBuilder tableDescriptorBuilder=
        TableDescriptorBuilder.newBuilder(TableName.valueOf(namespace,tableName));
        // 2.2 添加参数
        for(String columnFamily:columnFamilies){
        // 2.3 创建列族描述的建造者
        ColumnFamilyDescriptorBuilder columnFamilyDescriptorBuilder= ColumnFamilyDescriptorBuilder.newBuilder(Bytes.toBytes(columnFamily));
        // 2.4 对应当前的列族添加参数
        // 添加版本参数
        columnFamilyDescriptorBuilder.setMaxVersions(5);
        // 2.5 创建添加完参数的列族描述
        tableDescriptorBuilder.setColumnFamily(columnFamilyDescriptorBuilder.build());
        }
        // 2.6 创建对应的表格描述
        try{
        admin.createTable(tableDescriptorBuilder.build());
        }catch(IOException e){
        e.printStackTrace();
        }
        // 3. 关闭 admin
        admin.close();
        }
```

#### 修改表
```
/**
 * 修改表格中一个列族的版本
 *
 * @param namespace 命名空间名称
 * @param tableName 表格名称
 * @param columnFamily 列族名称
 * @param version 版本
 */
public static void modifyTable(String namespace,String tableName,String columnFamily,int version)throws IOException{
        // 判断表格是否存在
        if(!isTableExists(namespace,tableName)){
        System.out.println("表格不存在无法修改");
        return;
        }
        // 1. 获取 admin
        Admin admin=connection.getAdmin();
        try{
        // 2. 调用方法修改表格
        // 2.0 获取之前的表格描述
        TableDescriptor descriptor=admin.getDescriptor(TableName.valueOf(namespace,tableName));
        // 2.1 创建一个表格描述建造者
        // 如果使用填写 tableName 的方法 相当于创建了一个新的表格描述建造
        者 没有之前的信息
        // 如果想要修改之前的信息 必须调用方法填写一个旧的表格描述
        TableDescriptorBuilder tableDescriptorBuilder=TableDescriptorBuilder.newBuilder(descriptor);
        // 2.2 对应建造者进行表格数据的修改
        ColumnFamilyDescriptor columnFamily1=descriptor.getColumnFamily(Bytes.toBytes(columnFamily));
        // 创建列族描述建造者
        // 需要填写旧的列族描述
        ColumnFamilyDescriptorBuilder columnFamilyDescriptorBuilder=ColumnFamilyDescriptorBuilder.newBuilder(columnFamily1);
        // 修改对应的版本
        columnFamilyDescriptorBuilder.setMaxVersions(version);
        // 此处修改的时候 如果填写的新创建 那么别的参数会初始化

        tableDescriptorBuilder.modifyColumnFamily(columnFamilyDescriptorBuilder.build());
        admin.modifyTable(tableDescriptorBuilder.build());
        }catch(IOException e){
        e.printStackTrace();
        }
        // 3. 关闭 admin
        admin.close();
        }
```

#### 删除表
```
/**
 * 删除表格
 *
 * @param namespace 命名空间名称
 * @param tableName 表格名称
 * @return true 表示删除成功
 */
public static boolean deleteTable(String namespace,String tableName)throws IOException{
        // 1. 判断表格是否存在
        if(!isTableExists(namespace,tableName)){
        System.out.println("表格不存在 无法删除");
        return false;
        }
        // 2. 获取 admin
        Admin admin=connection.getAdmin();
        // 3. 调用相关的方法删除表格
        try{
        // HBase 删除表格之前 一定要先标记表格为不可以
        TableName tableName1=TableName.valueOf(namespace,tableName);
        admin.disableTable(tableName1);
        admin.deleteTable(tableName1);
        }catch(IOException e){
        e.printStackTrace();
        }
        // 4. 关闭 admin
        admin.close();
        return true;
        }
```

# DML语句

#### 插入数据
```
/**
 * 插入数据
 *
 * @param namespace 命名空间名称
 * @param tableName 表格名称
 * @param rowKey 主键
 * @param columnFamily 列族名称
 * @param columnName 列名
 * @param value 值
 */
public static void putCell(String namespace,String tableName,String rowKey,String columnFamily,String columnName,String value)throws IOException{
        // 1. 获取 table
        Table table=connection.getTable(TableName.valueOf(namespace,tableName));
        // 2. 调用相关方法插入数据
        // 2.1 创建 put 对象
        Put put=new Put(Bytes.toBytes(rowKey));
        // 2.2. 给 put 对象添加数据

        put.addColumn(Bytes.toBytes(columnFamily),Bytes.toBytes(columnNam e),Bytes.toBytes(value));
        // 2.3 将对象写入对应的方法
        try{
        table.put(put);
        }catch(IOException e){
        e.printStackTrace();
        }
        // 3. 关闭 table
        table.close();
        }
```

#### 读取数据
```
/**
 * 读取数据 读取对应的一行中的某一列
 *
 * @param namespace 命名空间名称
 * @param tableName 表格名称
 * @param rowKey 主键
 * @param columnFamily 列族名称
 * @param columnName 列名
 */
public static void getCells(String namespace,String tableName,String rowKey,String columnFamily,String columnName)throws IOException{
        // 1. 获取 table
        Table table=connection.getTable(TableName.valueOf(namespace,tableName));
        // 2. 创建 get 对象
        Get get=new Get(Bytes.toBytes(rowKey));
        // 如果直接调用 get 方法读取数据 此时读一整行数据
        // 如果想读取某一列的数据 需要添加对应的参数
        get.addColumn(Bytes.toBytes(columnFamily),Bytes.toBytes(columnName));
        // 设置读取数据的版本
        get.readAllVersions();
        try{
        // 读取数据 得到 result 对象
        Result result=table.get(get);
        // 处理数据
        Cell[]cells=result.rawCells();
        // 测试方法: 直接把读取的数据打印到控制台
        // 如果是实际开发 需要再额外写方法 对应处理数据
        for(Cell cell:cells){
        // cell 存储数据比较底层
        String value=new String(CellUtil.cloneValue(cell));
        System.out.println(value);
        }
        }catch(IOException e){
        e.printStackTrace();
        }
        // 关闭 table
        table.close();
        }

```

#### 扫描数据
```
/**
 * 扫描数据
 *
 * @param namespace 命名空间
 * @param tableName 表格名称
 * @param startRow 开始的 row 包含的
 * @param stopRow 结束的 row 不包含
 */
public static void scanRows(String namespace,String tableName,String startRow,String stopRow)throws IOException{
        // 1. 获取 table
        Table table=connection.getTable(TableName.valueOf(namespace,tableName));
        // 2. 创建 scan 对象
        Scan scan=new Scan();
        // 如果此时直接调用 会直接扫描整张表
        // 添加参数 来控制扫描的数据
        // 默认包含
        scan.withStartRow(Bytes.toBytes(startRow));
        // 默认不包含
        scan.withStopRow(Bytes.toBytes(stopRow));
        try{
        // 读取多行数据 获得 scanner
        ResultScanner scanner=table.getScanner(scan);
        // result 来记录一行数据 cell 数组
        // ResultScanner 来记录多行数据 result 的数组
        for(Result result:scanner){
        Cell[]cells=result.rawCells();
        for(Cell cell:cells){
        System.out.print(new String(CellUtil.cloneRow(cell))+"-"
        +new String(CellUtil.cloneFamily(cell))+"-"
        +new String(CellUtil.cloneQualifier(cell))+"-"
        +new String(CellUtil.cloneValue(cell))+"");
        }
        System.out.println();
        }
        }catch(IOException e){
        e.printStackTrace();
        }
        // 3. 关闭 table
        table.close();
        }
```

#### 带过滤扫描
```
/**
 * 带过滤的扫描
 *
 * @param namespace 命名空间
 * @param tableName 表格名称
 * @param startRow 开始 row
 * @param stopRow 结束 row
 * @param columnFamily 列族名称
 * @param columnName 列名
 * @param value value 值
 * @throws java.io.IOException
 */
public static void filterScan(String namespace,String tableName,String startRow,String stopRow,String columnFamily,String columnName,String value)throws IOException{
        // 1. 获取 table
        Table table=connection.getTable(TableName.valueOf(namespace,tableName));
        // 2. 创建 scan 对象
        Scan scan=new Scan();
        // 如果此时直接调用 会直接扫描整张表
        // 添加参数 来控制扫描的数据
        // 默认包含
        scan.withStartRow(Bytes.toBytes(startRow));
        // 默认不包含
        scan.withStopRow(Bytes.toBytes(stopRow));
        // 可以添加多个过滤
        FilterList filterList=new FilterList();
        // 创建过滤器
        // (1) 结果只保留当前列的数据
        ColumnValueFilter columnValueFilter=new ColumnValueFilter(
        // 列族名称
        Bytes.toBytes(columnFamily),
        // 列名
        Bytes.toBytes(columnName),
        // 比较关系
        CompareOperator.EQUAL,
        // 值
        Bytes.toBytes(value)
        );
        // (2) 结果保留整行数据
        // 结果同时会保留没有当前列的数据
        SingleColumnValueFilter singleColumnValueFilter=new SingleColumnValueFilter(
        // 列族名称
        Bytes.toBytes(columnFamily),
        // 列名
        Bytes.toBytes(columnName),
        // 比较关系
        CompareOperator.EQUAL,
        // 值
        Bytes.toBytes(value)
        );
        // 本身可以添加多个过滤器
        filterList.addFilter(singleColumnValueFilter);
       // 添加过滤
        scan.setFilter(filterList);
        try {
        // 读取多行数据 获得 scanner
        ResultScanner scanner = table.getScanner(scan);
        // result 来记录一行数据 cell 数组
        // ResultScanner 来记录多行数据 result 的数组
        for (Result result : scanner) {
        Cell[] cells = result.rawCells();
        for (Cell cell : cells) {
        System.out.print(new
        String(CellUtil.cloneRow(cell)) + "-" + new
        String(CellUtil.cloneFamily(cell)) + "-" + new
        String(CellUtil.cloneQualifier(cell)) + "-" + new
        String(CellUtil.cloneValue(cell)) + "\t");
        }
        System.out.println();
        }
        } catch (IOException e) {
        e.printStackTrace();
        }
        // 3. 关闭 table
        table.close();
        }
```

#### 删除数据
```
/**
 * 删除 column 数据
 *
 * @param nameSpace
 * @param tableName
 * @param rowKey
 * @param family
 * @param column
 * @throws java.io.IOException
 */
public static void deleteColumn(String nameSpace,String tableName,String rowKey,String family,String column)throws IOException{
        // 1.获取 table
        Table table=connection.getTable(TableName.valueOf(nameSpace,
        tableName));
        // 2.创建 Delete 对象
        Delete delete=new Delete(Bytes.toBytes(rowKey));
        // 3.添加删除信息
        // 3.1 删除单个版本
//
        delete.addColumn(Bytes.toBytes(family),Bytes.toBytes(column));
        // 3.2 删除所有版本
        delete.addColumns(Bytes.toBytes(family),
        Bytes.toBytes(column));
        // 3.3 删除列族
// delete.addFamily(Bytes.toBytes(family));
        // 3.删除数据
        table.delete(delete);
        // 5.关闭资源
        table.close();
        }
public static void main(String[]args)throws IOException{
        deleteColumn("bigdata","student","1001","info","name");
        }
```

## 