---
title: MapReduce的InputFormat切片规则具体实现五
date: 2022-08-01 16:20:14
tags: [大数据,hadoop,MapReduce]
---
# FileInputFormat切片原理

```
1.程序先找到你数据存储的目录

2.开始遍历处理(规划切片)下的每一个文件

3.遍历第一个文件ss.txt
   3.1 获取文件大小fs.sizeOf
   (ss.txt)
   3.2 计算切片大小 computeSplitSize(Math.max(minSize,Math.min(maxSize,blocksize)))=blocksize=123M
   3.3 默认情况下,切片大小=hdfs的块大小
   3.4 开始切,行程第一个切片:ss.txt-0:128M(每次切片时,都要判断切完剩下的部分是否大于块的1.1倍,不大于就不形成一个新的切片)
   3.5将切片信息写到一个切片规划文件中(HDFS输出路径中有个*.split的文件)
   3.6整个切片的核心过程在getSplit()方法中完成
   3.7 InputSplit只记录了切片的元数据信息,比如其实位置,长度,及其所在的节点列表等
   
4. 提交切片规范化文件到YARN上,YARN上的MrAppMatser就可以根据切片规划文件计算开启MapTask个数;

5.开始任务调度计算 
```
<!--more-->
## 获取切片信息大小
```
//获取切片的文件名称
String name=inputSplit.getPath().getName();

//根据文件类型获取切片信息
FileSplit inputSplit=(FileSplit) context.getInputSplit();
```


## FileInputFormat   切片机制
输入的对象是: 文件
基于行的日志文件,二进制格式文件,数据表等...

```
1.简单的按照文件的内容长度进行切分
2.切片大小,默认等于block大小
3.切片时不考虑数据集整体,而是逐个针对每一个文件单独切片

```
### 常见的实现类
```
1.TextInputFormat  (常用)  按行进行读取
2.KeyValueTextInputFormat   按分隔符切割 第0位为Key,第1位为value
3.NLineInputFormat   一次读取多行内容放在一起统一处理
4.CombineTextInputFormat  (常用) 主要处理多个小文件合并处理,一次读取多个文件,合并开启1个MapTask
5.自定义InputFormat等
```

## TextInputFormat  切片机制
`TextInputFormat`是默认的`FileInputFormat`实现类
输入的对象是:
按行 读取每条记录
```
key 对应偏移量 (String.length)
value 对应当前行的数据
```