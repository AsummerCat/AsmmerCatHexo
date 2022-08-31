---
title: SparkSql的数据的加载和保存四
date: 2022-08-31 14:29:44
tags: [大数据,Spark,SparkSql]
---
# SparkSql的数据的加载和保存
`spark.read.load` 是加载数据的通用方法
`SparkSQL 默认读取和保存的文件格式为 parquet`
例如:
```
spark.sql("select * from json.`/opt/module/data/user.json`").show
```

SaveMode 是一个枚举类，其中的常量包括：

| Scala/Java | Any Language | Meaning |
| --- | --- | --- |
| SaveMode.ErrorIfExists(default) | "error"(default) | 如果文件已经存在则抛出异常 |
|SaveMode.Append|"append"|如果文件已经存在则追加|
|SaveMode.Overwrite|"overwrite"|如果文件已经存在则覆盖|
|SaveMode.Ignore|"ignore"|如果文件已经存在则忽略|

修改保存模式
`df.write.mode("append").json("/opt/module/data/output")`
<!--more-->
# Parquet
Spark SQL 的默认数据源为 Parquet 格式。Parquet是一种能够有效存储嵌套数据的列式存储格式。

数据源为 Parquet 文件时，Spark SQL 可以方便的执行所有的操作，不需要使用 format。
修改配置项 spark.sql.sources.default，可修改默认数据源格式。

### 加载mysql数据

需要引入mysql驱动
```
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.49</version>
        </dependency>
```

```
  //1.读取配置
    val sparkConf = new SparkConf().setMaster("local[*]").setAppName("SparkSql")
    //2.创建SQLcontext对象
    val sqlc = SparkSession.builder().config(sparkConf).getOrCreate()
  //读取mysql数据
    //方式 1：通用的 load 方法读取
    sqlc.read.format("jdbc")
      .option("url", "jdbc:mysql://linux1:3306/spark-sql")
      .option("driver", "com.mysql.jdbc.Driver")
      .option("user", "root")
      .option("password", "123123")
      .option("dbtable", "user")
      .load().show
    
    //方式 2:通用的 load 方法读取 参数另一种形式
    sqlc.read.format("jdbc")
      .options(Map("url" -> "jdbc:mysql://linux1:3306/spark-sql?user=root&password=123123", "dbtable" -> "user", "driver" -> "com.mysql.jdbc.Driver")).load().show
    
    //方式 3:使用 jdbc 方法读取
    val props: Properties = new Properties()
    props.setProperty("user", "root")
    props.setProperty("password", "123123")
    val df: DataFrame = sqlc.read.jdbc("jdbc:mysql://linux1:3306/spark-sql", "user", props)
    df.show
```
## 读取文件
```
package day6

import org.apache.spark.SparkConf
import org.apache.spark.sql.{DataFrame, SparkSession}

import java.util.Properties

/**
 * SparkSql 数据的加载和保存
 */
object SparkLoadFile {
  def main(args: Array[String]): Unit = {
    //1.读取配置
    val sparkConf = new SparkConf().setMaster("local[*]").setAppName("SparkSql")
    //2.创建SQLcontext对象
    val sqlc = SparkSession.builder().config(sparkConf).getOrCreate()

    //直接加载数据 并且转换为DataFrame
    sqlc.sql("select * from json.`/opt/module/data/user.json`").show

    //加载数据
    val frame = sqlc.read.json("xxx/1.json")
    //保存数据  ->保存为 parquet 格式
    frame.write.mode("append").save("/opt/module/data/output")


    //读取mysql数据
    //方式 1：通用的 load 方法读取
    sqlc.read.format("jdbc")
      .option("url", "jdbc:mysql://linux1:3306/spark-sql")
      .option("driver", "com.mysql.jdbc.Driver")
      .option("user", "root")
      .option("password", "123123")
      .option("dbtable", "user")
      .load().show

    //方式 2:通用的 load 方法读取 参数另一种形式
    sqlc.read.format("jdbc")
      .options(Map("url" -> "jdbc:mysql://linux1:3306/spark-sql?user=root&password=123123", "dbtable" -> "user", "driver" -> "com.mysql.jdbc.Driver")).load().show

    //方式 3:使用 jdbc 方法读取
    val props: Properties = new Properties()
    props.setProperty("user", "root")
    props.setProperty("password", "123123")
    val df: DataFrame = sqlc.read.jdbc("jdbc:mysql://linux1:3306/spark-sql", "user", props)
    df.show

    //释放资源
    sqlc.stop()


  }
}

```

## 写入文件
```
package day6

import org.apache.spark.SparkConf
import org.apache.spark.rdd.RDD
import org.apache.spark.sql.{Dataset, SaveMode, SparkSession}

import java.util.Properties

/**
 * SparkSql 数据的保存
 */
object SparkSaveFile {
  def main(args: Array[String]): Unit = {
    //1.读取配置
    val sparkConf = new SparkConf().setMaster("local[*]").setAppName("SparkSql")
    //2.创建SQLcontext对象
    val sqlc = SparkSession.builder().config(sparkConf).getOrCreate()
    //3.导入隐式转换包 后面很多操作都需要它
    import sqlc.implicits._

    val rdd: RDD[User] = sqlc.sparkContext.makeRDD(List(User("lisi", 20), User("zs", 30)))
    val ds: Dataset[User] = rdd.toDS

    //保存为其他格式 format指定为json
    val frame = rdd.toDF()
    frame.write.format("json").save("oupuut1")

    //方式 1：通用的方式 format 指定写出类型
    ds.write
      .format("jdbc")
      .option("url", "jdbc:mysql://linux1:3306/spark-sql")
      .option("user", "root")
      .option("password", "123123")
      .option("dbtable", "user")
      .mode(SaveMode.Append)
      .save()

    //方式 2：通过 jdbc 方法
    val props: Properties = new Properties()
    props.setProperty("user", "root")
    props.setProperty("password", "123123")
    ds.write.mode(SaveMode.Append).jdbc("jdbc:mysql://linux1:3306/spark-sql",
      "user", props)
    //释放资源
    sqlc.stop()


  }

  case class User(var name: String, var age: Long)

}

```