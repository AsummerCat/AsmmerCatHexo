---
title: SparkSql的基础使用二
date: 2022-08-31 14:28:53
tags: [大数据,Spark,SparkSql]
---
# SparkSql的基础使用

## 导入对应pom
注意 版本spark-sql的版本要和spark-core版本一致
```
    <dependencies>
        <dependency>
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-core_2.12</artifactId>
            <version>3.0.3</version>
        </dependency>
        <dependency>
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-sql_2.12</artifactId>
            <version>3.0.3</version>
        </dependency>
    </dependencies>
```

<!--more-->

## 基础语法
```
    //1.读取配置
    val sparkConf = new SparkConf().setMaster("local[*]").setAppName("SparkSql")
    //2.创建SQLcontext对象
    val sqlc = SparkSession.builder().config(sparkConf).getOrCreate()
    //3.导入隐式转换包 后面很多操作都需要它
    import sqlc.implicits._
```

## 编写代码
```
package day5

import org.apache.spark.SparkConf
import org.apache.spark.sql.{Dataset, SparkSession}

/**
 * 使用sparkSql 基础语法
 *
 * DataFrame
 * DataSet
 */
object SparkSQLDemo {
  def main(args: Array[String]): Unit = {
    //1.读取配置
    val sparkConf = new SparkConf().setMaster("local[*]").setAppName("SparkSql")
    //2.创建SQLcontext对象
    val sqlc = SparkSession.builder().config(sparkConf).getOrCreate()
    //3.导入隐式转换包 后面很多操作都需要它
    import sqlc.implicits._

    //4.获取数据源
    val df = sqlc.read.json("SparkWordCount\\src\\data\\user.json")

    //5. 显示表格
    df.show()

    //6.使用 DataFrame =>Sql
    //创建一个临时视图
    df.createOrReplaceTempView("user")
    //编写sql语句
    sqlc.sql("select name,age from user").show()


    //7.使用 DataFrame =>Dql
    df.select("age", "name").show()

    //8. DataSet
    val dataSet: Dataset[String] = sqlc.read.json("SparkWordCount\\src\\data\\user.json").toJSON
    dataSet.show();

    //9.RDD 转换 DataFrame
    val rdd = sqlc.sparkContext.makeRDD(List((1, "小明", 21), (2, "小明", 22), (3, "小明", 23), (4, "小明", 24)))
    //DataFrame 转换指定位置的字段名称
    val frame = rdd.toDF("age", "name", "address")
    frame.show()

    //10.DataFrame 转换 DataSet (需要有个转换类)
    val data = frame.as[User]
    data.show()

    //11. DataSet 转换 DataFrame
    val frame1 = data.toDF()
    frame1.show()


    //12. RDD转换DataSet
    val value = rdd.map {
      case (age, name, address) =>
        User(age, name, address)
    }.toDS()

    value.show()

    //13.DataSet转换RDD
    val userRdd = value.rdd

    //最后关闭连接
    sqlc.close();
  }


  case class User(age: Int, name: String, address: Int)
}


```
