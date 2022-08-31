---
title: Spark的RDD持久化缓存八
date: 2022-08-31 14:06:46
tags: [大数据,Spark]
---
# Spark的RDD持久化缓存
![image](/img/2022-08-24/1.png)
对象可重用,但是数据无法重用,
所以以上思路是错误的
<!--more-->
所以会有个RDD持久化缓存的概念存在

# 缓存 (持久化) cache
```
rdd1.cache();

对应RDD.cacahe();

就相当于给这个位置做了缓存,打个一个断点


 //2.加入缓存节点
    rdd1.cache();
    //或者使用
    rdd1.persist();
    
*.chae() 默认持久化操作,只能将数据保存到内存中
如果想要保存磁盘文件或者其他介质中,可用
*.persist(StorageLevel.MEMORY_ONLY);
持久化操作必须在 collect()之后才会产生数据

```

# 设置检查点 checkpoint
job执行完毕后,检查点不会自动删除,而cache会自动删除
流程:
1.设置检查点保存路径
2.使用检查点

<font color='red'>为了提高效率,一般是cache和checkpoint一起使用</font>
```
package day3

import org.apache.spark.{SparkConf, SparkContext}

/**
 * RDD 设置检查点
 */
object Spark03_RDD_checkPoint {
  def main(args: Array[String]): Unit = {

    //创建scala和Spark框架的链接
    val sparkConf = new SparkConf().setMaster("local[*]").setAppName("WordCount")
    val sc = new SparkContext(sparkConf)

    //1.设置检查点保存路径
    sc.setCheckpointDir("SparkWordCount\\src\\cache")

    //执行业务逻辑
    //2.创建RDD 并行RDD(业务列表,调用CPU核数分片处理)
    val rdd1 = sc.parallelize(List(1, 2, 3, 4, 5), 2)

    //默认执行逻辑计算逻辑
    val rdd2 = rdd1.map(num=>{
      println("xxxxxxxxx")
      num*1
    })

    //3.设置检查点
    //需要落盘,需要指定检查点保存路径
    //cache()在job执行完毕后会自动删除,而 检查点在job执行完毕后还存在
    rdd1.cache();
    rdd2.checkpoint();

    //计算逻辑1
    val value = rdd2.map(num=>{
      num*2
    })
    value.collect().foreach(print)
    println("")
    println("****************************")

    //计算逻辑2
    val value1 = rdd2.map(num=>{
      println("xxxxxxxxx")
      num*3
    })
    value1.collect().foreach(print)

    //关闭链接
    sc.stop()
  }
}
```
 