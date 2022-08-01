---
title: Yarn代码自定义参数调用八
date: 2022-08-01 16:16:14
tags: [大数据,hadoop,yarn]
---
# Yarn代码自定义参数调用
实现Tool接口,动态修改参数

## 代码层级
```
1. mian 函数启动类

2. job实现tool接口类

3.匿名 MapTask和ReduceTask类
```
## 在job类上实现 Tool接口 及其匿名任务类
```
import org.apache.hadoop.util.Tool; 
```
<!--more-->
案例:
```
package com.hadoop.mapreduce;
import org.apache.hadoop.util.Tool; 
 
import java.io.IOException;
 
public class Runner implements Tool{
   private Configuration conf;
   
   @Override
    public int run(String[] args) throws Exception{
       Job job=Job.getInstance(conf);
       //以下部分就是正常的job编写
        return 0;
    }
    
    @Override
    public void setConf(Configuration conf) {
     this.conf=conf;
    }
    @Override    
    public Configuration getConf(){
        return conf;
    }
    
    //以下部分使用匿名内部类
    实现MapTask和ReduceTask
}
```

## 编写main函数调用类
```
public class dorunDriver{
    
    private static Tool tool;
    public static void main(String[] args){
        //创建配置
        Configuration conf=new Configuration();
        
        switch(args[0]){
            case "wordcount":
                 //指定参数运行某个任务
                 tool=new Runner();
                 break;
            default:
                 throw new RuntimeException("no such tool"+args[0]);
        }
        //执行任务
        //参数: 配置信息, 具体任务,参数(使用了args[0],所以从1开传递) 
        TooRunner.run(conf,tool,Arrays.CopyOfRange(1,args.length));
        
        System.exit(run);
    }
}
```

# Tool实现三个方法
```
//核心驱动类 (conf 需要传入)
public int run(String[] args)

//设置一个配置信息
public void setConf(Configuration conf)

//获取一个配置信息
public Configuration getConf()
```
