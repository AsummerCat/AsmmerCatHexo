---
title: SparkSql自定义函数扩展三
date: 2022-08-31 14:29:21
tags: [大数据,Spark,SparkSql]
---
# SparkSql自定义函数扩展


# UDF 函数 创建自定义函数扩展
## 例子:
类似mysql的那种函数
`sqlc.udf.register("addName",(x:String)=> "Name:"+x)`

```
package day5

import org.apache.spark.SparkConf
import org.apache.spark.sql.SparkSession

/**
 * sprakSql自定义函数扩展
 */
object SparkSqlCustomFun {
  def main(args: Array[String]): Unit = {
    //1.读取配置
    val sparkConf = new SparkConf().setMaster("local[*]").setAppName("SparkSql")
    //2.创建SQLcontext对象
    val sqlc = SparkSession.builder().config(sparkConf).getOrCreate()
    //3.导入隐式转换包 后面很多操作都需要它

    //4.获取数据源
    val df = sqlc.read.json("SparkWordCount\\src\\data\\user.json")

    //5.创建临时表
    df.createOrReplaceTempView("user")

    //6.创建udf自定义函数 并注册
    sqlc.udf.register("addName", (x: String) => "Name:" + x)

    //7.编写查询语句并且加入自定义函数
    sqlc.sql("Select addName(name) as name,age from user").show()


    //关闭连接
    sqlc.close();
  }
}

```
<!--more-->

# UDAF函数 弱类型  2.x的语法

`弱类型表示 根据值的顺序位置来处理`
`定义类继承 UserDefinedAggregateFunction，并重写其中方法`
## 例子:
```
package day5

import org.apache.spark.SparkConf
import org.apache.spark.sql.expressions.{MutableAggregationBuffer, UserDefinedAggregateFunction}
import org.apache.spark.sql.types._
import org.apache.spark.sql.{Row, SparkSession}

/**
 * sprakSql自定义函数扩展  UDAF弱类型 按照顺序位来操作
 */
object SparkSqlCustomFunUDAF {
  def main(args: Array[String]): Unit = {
    //1.读取配置
    val sparkConf = new SparkConf().setMaster("local[*]").setAppName("SparkSql")
    //2.创建SQLcontext对象
    val sqlc = SparkSession.builder().config(sparkConf).getOrCreate()
    //3.导入隐式转换包 后面很多操作都需要它

    //4.获取数据源
    val df = sqlc.read.json("SparkWordCount\\src\\data\\user.json")

    //5.创建临时表
    df.createOrReplaceTempView("user")

    //6.创建udf自定义函数 并注册
    //创建聚合函数 求平均
    val myAverage = new MyAveragUDAF
    //在 spark 中注册聚合函数
    sqlc.udf.register("avgAge", myAverage)


    //7.编写查询语句并且加入自定义函数
    sqlc.sql("Select avgAge(age) from user").show()


    //关闭连接
    sqlc.close();
  }


  /*
定义类继承 UserDefinedAggregateFunction，并重写其中方法
*/
  class MyAveragUDAF extends UserDefinedAggregateFunction {
    // 聚合函数输入参数的数据类型
    def inputSchema: StructType = StructType(Array(StructField("age", IntegerType)))

    // 聚合函数缓冲区中值的数据类型(age,count)
    def bufferSchema: StructType = {

      StructType(Array(StructField("sum", LongType), StructField("count", LongType)))
    }

    // 函数返回值的数据类型
    def dataType: DataType = DoubleType

    // 稳定性：对于相同的输入是否一直返回相同的输出。
    def deterministic: Boolean = true

    // 函数缓冲区初始化
    def initialize(buffer: MutableAggregationBuffer): Unit = {
      // 存年龄的总和
      buffer(0) = 0L
      // 存年龄的个数
      buffer(1) = 0L
    }

    // 更新缓冲区中的数据
    def update(buffer: MutableAggregationBuffer, input: Row): Unit = {
      if (!input.isNullAt(0)) {
        buffer(0) = buffer.getLong(0) + input.getInt(0)
        buffer(1) = buffer.getLong(1) + 1
      }
    }

    // 合并缓冲区
    def merge(buffer1: MutableAggregationBuffer, buffer2: Row): Unit = {
      buffer1(0) = buffer1.getLong(0) + buffer2.getLong(0)
      buffer1(1) = buffer1.getLong(1) + buffer2.getLong(1)
    }

    // 计算最终结果
    def evaluate(buffer: Row): Double = buffer.getLong(0).toDouble / buffer.getLong(1)
  }


}

```

# # UDAF函数 强类型
根据属性来操作
`定义类继承 org.apache.spark.sql.expressions.Aggregator，并重写其中方法`

```
package day5

import org.apache.spark.SparkConf
import org.apache.spark.sql.expressions.Aggregator
import org.apache.spark.sql.{Encoder, Encoders, SparkSession, functions}

/**
 * sprakSql自定义函数扩展  UDAF强类型 按照属性来操作
 */
object SparkSqlCustomFunUDAF1 {
  def main(args: Array[String]): Unit = {
    //1.读取配置
    val sparkConf = new SparkConf().setMaster("local[*]").setAppName("SparkSql")
    //2.创建SQLcontext对象
    val sqlc = SparkSession.builder().config(sparkConf).getOrCreate()
    //3.导入隐式转换包 后面很多操作都需要它

    //4.获取数据源
    val df = sqlc.read.json("SparkWordCount\\src\\data\\user.json")

    //5.创建临时表
    df.createOrReplaceTempView("user")

    //6.创建udf自定义函数 并注册
    //在 spark 中注册聚合函数 求平均
    sqlc.udf.register("avgAge", functions.udaf(new MyAveragUDAF()))

    //7.编写查询语句并且加入自定义函数
    sqlc.sql("Select avgAge(age) from user").show()



    //关闭连接
    sqlc.close();
  }


  /*
   * 定义类继承 org.apache.spark.sql.expressions.Aggregator，并重写其中方法
   * IN: 输入数据类型
   * BUF: 缓冲区 中间处理的数据类型
   * OUT: 输出的数据类型
   */
  case class Buff(var total: Long, var count: Long)

  class MyAveragUDAF extends Aggregator[Long, Buff, Long] {
    //初始值 缓冲区的初始化
    override def zero: Buff = {
      Buff(0L, 0L)
    }

    //根据输入的数据来更新缓冲区的额数据
    override def reduce(b: Buff, a: Long): Buff = {
      b.total = b.total + a
      b.count = b.count + 1
      b
    }

    //合并缓冲区
    override def merge(b1: Buff, b2: Buff): Buff = {
      b1.count = b1.count + b2.count
      b1.total = b1.total + b2.total
      b1
    }

    //计算结果
    override def finish(reduction: Buff): Long = {
      reduction.total / reduction.count

    }

    //缓冲区的编码操作
    override def bufferEncoder: Encoder[Buff] = Encoders.product

    //输出的编码操作
    override def outputEncoder: Encoder[Long] = Encoders.scalaLong
  }

}

```