---
title: MapReduce的Join操作多表关联输出
date: 2022-08-01 16:23:56
tags: [大数据,hadoop,MapReduce]
---
# MapReduce的Join操作多表关联输出
分为`MapJoin`和`ReduceJoin`

```
实现效果:
就是数据库的join 查询
输出的文件中 对应的关系建立

例子:
根据商品信息表中的商品pid合并到订单数据表中
```
<!--more-->

## Reduce的Join

`Reduce端表合并(数据倾斜)`
缺点: 合并的操作是在`ReduceTask`上进行合并操作的,可能60个MapTask的数据汇聚在同一个`ReduceTask`上,资源资源利用率不高,效率低; 推荐在`MapTask`上进行join操作

## Map的Join
Map端实现数据合并
这里的操作->
```
1.先把小表汇聚到内存中
2.大表遍历的时候->进行跟小表(HashMap)匹配,
匹配完成后直接发送`ReduceTask`减轻压力
```


## Reduce的Join操作流程

1. 建立一个自定义的`Writable`,实现自定义序列化
   (字段包含: 商品id,商品价格,商品名称<关联的字段>,来源表)

2.  `MapTask`中使用关键字段作为key`商品id`
    编写一个`setup()`方法进行前置处理,区分输出

3.  `ReduceTask`中因为key是商品id,
    value是具体内容的list;
    进行类拷贝将数据缺失的内容构建成对应的一条数据
    输出即可;


## Map的join操作流程
采用 `DistributedCache`

0. 首先需要`Writable`自定义序列化 或者输出的时候直接 String直接拼接

1.在`Mapper`的`setup`阶段,将文件读取到缓存集合中

2.在job类上加载缓存
```
//缓存普通文件到Task运行节点上
job.addCacheFile(new URI("file:///e:/cache/pd.txt"));
//如果是集群运行,需要设置HDFS路径
job.addCacheFile(new URI("hdfs://ip:8020/cache/pd.txt"));
```

3. Map端join的逻辑不需要`Reduce`阶段
```
设置ReduceTask数量为0

job.setNumReduceTasks(0);
```

4. 在`setup()`阶段进行读取缓存数据
```

全局变量 Map<String,String> cacheMap=new HashMap<>();

void setup(Context context){
    //获取缓存的问题,并发文件内容封装到集合
    URI[] cacheFiles =context.getCacheFiles();
    
    FileSystem fs =FileSystem.get(context.getConfiguration());
    FSDataInputStream fis=fs.ipen(new Path(cacheFiles[0]));
    
    //从流中获取数据
    BufferedReader reder=new BufferedReader(new InputStreamReader(fis."UTF-8"));
    
    String line;
    while (StringUtils.insNotEmpty(line=reader.readLine())){
        //切割行数据
        String[] fields =line.split("\t");
        
        赋值给全局变量作为缓存使用
        cacheMap.put(fields[0],fields[1]);
    }
    
    //最后关闭流
    IOUtils.closeStream(reader);
    
}

```

5. Map阶段照常读取数据
```
中间需要的操作就是

需要对比cacheMap中的数据


```

