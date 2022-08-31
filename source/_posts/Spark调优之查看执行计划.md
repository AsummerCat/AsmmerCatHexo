---
title: Spark调优之查看执行计划
date: 2022-08-31 14:34:16
tags: [大数据,Spark,SparkSql]
---
# Spark调优之查看执行计划

## 基本语法
`.explain(mode="xxx")`
```
从 3.0 开始，explain 方法有一个新的参数 mode，该参数可以指定执行计划展示格式： ➢ explain(mode="simple")：只展示物理执行计划。
➢ explain(mode="extended")：展示物理执行计划和逻辑执行计划。
➢ explain(mode="codegen") ：展示要 Codegen 生成的可执行 Java 代码。
➢ explain(mode="cost")：展示优化后的逻辑执行计划以及相关的统计。
➢ explain(mode="formatted")：以分隔的方式输出，它会输出更易读的物理执行计划，
并展示每个节点的详细信息。
```
<!--more-->

## 案例
```
package day9

import org.apache.spark.SparkConf
import org.apache.spark.sql.SparkSession

/**
 * spark 查看执行计划
 *
 */
object SparkExplain {
  def main(args: Array[String]): Unit = {
    //1.读取配置
    val sparkConf = new SparkConf().setMaster("local[*]").setAppName("SparkSql")
    //2.创建SQLcontext对象
    val sqlc = SparkSession.builder().config(sparkConf).getOrCreate()
    //3.导入隐式转换包 后面很多操作都需要它

    //4.获取数据源
    val df = sqlc.read.json("SparkWordCount\\src\\data\\user.json")

    //5. 显示表格
    df.show()

    //6.使用 DataFrame =>Sql
    //创建一个临时视图
    df.createOrReplaceTempView("user")
    //编写sql语句
    sqlc.sql("select name,age from user").explain()

    println("=====================================explain()-只展示物理执行计划============================================")
    sqlc.sql("select name,age from user").explain()

    println("===============================explain(mode = \"simple\")-只展示物理执行计划=================================")
    sqlc.sql("select name,age from user").explain(mode = "simple")

    println("============================explain(mode = \"extended\")-展示逻辑和物理执行计划==============================")
    sqlc.sql("select name,age from user").explain(mode = "extended")

    println("============================explain(mode = \"codegen\")-展示可执行java代码===================================")
    sqlc.sql("select name,age from user").explain(mode = "codegen")

    println("============================explain(mode = \"formatted\")-展示格式化的物理执行计划=============================")
    sqlc.sql("select name,age from user").explain(mode = "formatted")

    //最后关闭连接
    sqlc.close();
  }


  case class User(age: Int, name: String, address: Int)
}

```