---
title: Flink基础一
date: 2022-09-19 16:54:16
tags: [大数据,Flink]
---
# Flink基础

流处理和批处理一体化的东西

官网 https://flink.apache.org

## 注意
在Flink的架构里 一切都是流

有边界的数据 -> 批数据 -> 有界数据流
无边界的数据 -> 实时流数据 -> 无界数据流
`DataStream流`

[demo地址](https://github.com/AsummerCat/flinkdemo.git)
<!--more-->


## 导入Flink依赖
ps: 一定要注意 版本要一致
```
    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
        <flink.version>1.14.5</flink.version>
        <target.java.version>1.8</target.java.version>
        <scala.binary.version>2.12</scala.binary.version>
    </properties>
    <dependencies>
        <!-- 引入 Flink 相关依赖-->
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-scala_${scala.binary.version}</artifactId>
            <version>${flink.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-streaming-scala_${scala.binary.version}</artifactId>
            <version>${flink.version}</version>
        </dependency>
        <!--引入这个可以再本地执行flink命令-->
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-clients_${scala.binary.version}</artifactId>
            <version>${flink.version}</version>
        </dependency>
    </dependencies>
```

# 快速使用 -> 批数据处理
```
package day1

import org.apache.flink.api.scala.{ExecutionEnvironment, createTypeInformation}

object BatchWordCount {
  def main(args: Array[String]): Unit = {
    //1.创建执行环境并配置并行度
    val env = ExecutionEnvironment.getExecutionEnvironment
    //2.读取文本文件 ->DataSet格式
    val lineDS = env.readTextFile("input/words.txt")
    //3.对数据进行格式转换
    val wordAndOne = lineDS.flatMap(_.split(" ")).map(r => (r, 1))
    //4.对数据进行分组
    val wordAndOneUG = wordAndOne.groupBy(0)

    val value = wordAndOneUG.sum(1)
    // 5.打印输出
    value.print
  }
}
```



# 快速使用->有界流数据处理
```
package day1

import org.apache.flink.streaming.api.scala._

/**
 * streaming有界数据流 flink基础语法
 */
object BoundedSteamWordCount {
  def main(args: Array[String]): Unit = {
    //1.创建一个流式执行环境
    val environment = StreamExecutionEnvironment.getExecutionEnvironment

    //2.读取文本文件数据 获取DataStream
    val lineDataStream = environment.readTextFile("input/words.txt")

    //3.对数据进行格式转换
    val wordAndOne = lineDataStream.flatMap(_.split(" ")).map(r => (r, 1))
    //4.对数据进行分组
    val wordAndOneUG = wordAndOne.keyBy(_._1)

    val value = wordAndOneUG.sum(1)
    // 5.打印输出
    value.print

    //6.执行任务 注意流数据处理需要执行调度任务
    environment.execute()
  }


}
```

# 快速使用->无界流数据处理
```
package day1

import org.apache.flink.api.java.utils.ParameterTool
import org.apache.flink.streaming.api.scala._

/**
 * streaming无界数据流 flink基础语法
 */
object NoBoundedSteamWordCount {
  def main(args: Array[String]): Unit = {
    //1.创建一个流式执行环境
    val environment = StreamExecutionEnvironment.getExecutionEnvironment
    
    //1.1 模拟从main启动类参数中获取参数 flink提供的API
    val hostName = ParameterTool.fromArgs(args).get("hostName")
    val port = ParameterTool.fromArgs(args).get("port")

    //2.模拟监听端口发送数据 表示获取实时的数据 ->无界数据流
    val lineDataStream = environment.socketTextStream(hostName, port)

    //3.对数据进行格式转换
    val wordAndOne = lineDataStream.flatMap(_.split(" ")).map(r => (r, 1))

    //4.对数据进行分组
    val wordAndOneUG = wordAndOne.keyBy(_._1)

    val value = wordAndOneUG.sum(1)
    // 5.打印输出
    value.print

    //6.执行任务 注意流数据处理需要执行调度任务
    environment.execute()
  }
}
```


## 提交就业任务处理
```
$ bin/flink run -Dexecution.runtime-mode=BATCH BatchWordCount.jar
```