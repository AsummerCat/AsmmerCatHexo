---
title: MapReduce的Writable自定义序列化三
date: 2022-08-01 16:19:05
tags: [大数据,hadoop,MapReduce]
---
# MapReduce的Writable自定义序列化
## 重写 Writable接口

对应的
 ```
 write方法 (序列化)
 输出
 
 
 readFields方法 (反序列化)
 输入 
 ```
 <!--more-->

## 序列化步骤
实现一个Bean对象序列化过程 如下7步

#### 1.必须实现writable接口


#### 2.反序列化时,需要反射调用空参构造函数,所以必须有空参构造
 ```
 public FlowBean(){
     super();
 }
 ```
#### 3.重写序列化方法
 ```
 @Override
 public void write(DataOutput out) throws IOException
 {
 //这里是对应bean里的属性
     out.writeLong(upFlow);
     out.writeLong(downFlow);
     out.writeLong(sumFlow);
 }
 ```

#### 4.重写反序列化方法
 ```
  @Override
 public void readFields(DataIntput in) throws IOException
 {
   upFlow=in.readLong();
   downFlow=in.readLong();
   sumFlow=in.readLong();
 }
 
 ```

#### 5. 注意反序列化的顺序和序列化的顺序完全一致

##### 6. 要想把结果显示在文件中,需要重写toString(),可用`\t`分开,方便后续使用

##### 7.如果需要将自定义的bean放到key中传输,则还需要实现`WritableComparable`接口,因为`MapReduce`框中的`Shuffle`过程要求对key必须能排序
 ```
 @override
 public int compareTo(FlowBean O){
     //倒序排序,从大到小
     return this.sumFlow > O.getSumFlow()? -1:1;
 }
 
 ```