---
title: MapReduce基础调用案例二
date: 2022-08-01 16:18:36
tags: [大数据,hadoop,MapReduce]
---
# MapReduce基础调用案例

## 依赖注入
```
<dependency>
    <groupId>org.apache.hadoop</groupId>
    <artifactId>hadoop-client</artifactId>
    <version>3.3.1</version>
</dependency>
```
## 注意事项
在Windows平台上 下载文件错误
提示错误: 找不到winutils.exe 和 HADOOP_HOME没有设置

解: 需要本地库支撑
Hadoop.dll 和 winutils.exe
## 案例
<!--more-->

### 开发注意事项
Mapper的引入
```
1.x版本: org.apache.hadoop.mapred

2.x和3.x版本: org.apache.hadoop.mapreduce

```
### Mapper程序
`context.write`表示输出
```

public class TestMapper extends Mapper<LongWritable, Text,Text,LongWritable> {
    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
        String line = value.toString();//先将Text类型转换为String
        String[] words = line.split(" ");//对每一行的数据以空格分割
        for(String word:words){
            context.write(new Text(word),new LongWritable(1));//将每一个单词以(word,1)的形式写出去。
        }
    }

```

### reduce程序
```

public class TestRuduce extends Reducer<Text, LongWritable,Text,LongWritable> {
    @Override
    protected void reduce(Text key, Iterable<LongWritable> values, Context context) throws IOException, InterruptedException {
        long num = 0;
        Iterator<LongWritable> iterator = values.iterator();
        while(iterator.hasNext()){
            num=num+iterator.next().get();//将每个迭代的加起来，就可以得出总的单词个数
        }
        System.out.println(key+"   "+num);
        context.write(key,new LongWritable(num));
    }

```

### 主函数
```
package com.hadoop.mapreduce;
 
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
 
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
 
import java.io.IOException;
 
public class Runner {
    //这个类就是程序的入口了
    public static void main(String[] args) {
        Configuration conf = new Configuration();
        conf.set("fs.defaultFS","hdfs://192.168.40.138:9000");
 
        //新建1一个job任务
        try {
            Job job = Job.getInstance(conf);
            //设置jar包
            job.setJarByClass(Runner.class);
            //指定mapper和reducer
            job.setMapperClass(TestMapper.class);
            job.setReducerClass(TestRuduce.class);
 
            //设置map端输出类型
            job.setMapOutputKeyClass(Text.class);
            job.setMapOutputValueClass(LongWritable.class);
            //设置reduce的输出类型
            job.setOutputKeyClass(Text.class);
            job.setOutputValueClass(LongWritable.class);
 
            //指定mapreduce程序的输入和输出路径
            Path inputPath = new Path("/wordcount/input");//这里是hdfs的路径，首先要将input这个文件传到hdfs上去
            Path outputPath = new Path("/wordcount/output");//这里也是hdfs路径，我们查看结果也需要在hdfs上查看。
            FileInputFormat.setInputPaths(job,inputPath);//设置输入路径，这里的输入路径和输出路径都import org.apache.hadoop.mapreduce.lib.input下的，一定不要使用imapred下的，那是老包了，不要使用。
            FileOutputFormat.setOutputPath(job,outputPath);//设置输出路径
 
            //提交任务 true 监控并打印job信息
            boolean waitForCompletion = job.waitForCompletion(true);
            //成功:0 失败:1
            System.exit(waitForCompletion?0:1);
        } catch (IOException e) {
            e.printStackTrace();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
 
    }
}
```


# 调用集群运行
如果不写conf.set 表示本地运行
```
Configuration conf = new Configuration();
        conf.set("fs.defaultFS","hdfs://192.168.40.138:9000");

```
ps: 服务器运行,需要打包jar 上传到hadoop相关内网机子里

## 输出路径需要不写死,根据参数调用使用的话
```
main函数后面的参数 可以使用
main(String[] args)


args[0]  ,rgs[1] 等 

```

## 集群调用命令
```

hadoop jar xxx.jar com..xxx.Runner /input /out



命令详解:

运行jar   指定启动全类名  配置多个参数 

```
