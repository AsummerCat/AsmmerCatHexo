---
title: MapReduce的压缩
date: 2022-08-01 16:24:30
tags: [大数据,hadoop,MapReduce]
---
# MapReduce的压缩
## 压缩原则
1.运算密集型的job,少用压缩
2.IO密集型的job,多用压缩

<!--more-->

| 压缩格式 | Hdoop自带 | 算法 | 文件扩展名 | 是否可切片 | 压缩后,原程序是否需要修改 |
| --- | --- | --- | --- | --- | --- |
| DEFLATE |  是,直接使用|DEFLATE  | .deflate | 否 | 和文本处理一样,不需要修改 |
| Gzip | 是,直接使用 | DEFLATE | .gz | 否 |  和文本处理一样,不需要修改|
| bzip2 |  是,直接使用| bzip2 | .bz2 |  是| 和文本处理一样,不需要修改 |
| LZO | 否,需要安装 | LZO | .lzo | 是 | 需要建索引,还需要指定输入格式 |
| Snappy | 是,直接使用 | Snappy | .snappy |否  | 和文本处理一样,不需要修改 |

Snappy 生产大量使用
LZO 数仓可用
```
1.数据量小于快大小,重点考虑压缩和解压缩速度比较快的LZO/snappy

2.数据量非常大,重点考虑支持切片的Bzip2和LZO
```

## 何时做压缩?
`MapReduce`全过程中任意阶段启用;
```
1.MapTask 输入输出
2.ReduceTask 输入输出
```

## Haddop如何启用压缩
1.支持多种压缩/解压缩算法,Hadoop引入了编码/解码器

| 压缩格式 | 对应编码/解码器 |
| --- | --- |
|  DEFLATE| org.apache.hadoop.io.compress.DefaultCodec|
|  gzip| org.apache.hadoop.io.compress.GzipCodec |
|  bzip2| org.apache.hadoop.io.compress.BZip2Codec |
|  LZO| com.hadoop.compression.lzo.LzopCodec |
|  Snappy| org.apache.hadoop.io.compress.SnappyCodec |

2.要在Hadoop中启用压缩,可以配置如下参数


| 参数 | 默认值 | 阶段 | 建议 |
| --- | --- | --- | --- |
| io.compression.codecs(在core-site.xml中配置) |无,这个需要命令行输入hadoop checknative查看(查看hadoop支持的格式)  |输入压缩  |  Hadoop使用文件扩展名判断是否支持某种编解码器|
|mapreduce.map.output.compress(在mapred-site.xml中配置)|false|mapper输出|这个参数设为`true`启用压缩|
|mapreduce.map.output.compress.codec(在mapred-site.xml中配置)|org.apache.hadoop.io.compress.DefaultCodec|mapper输出|企业多使用LZO或Snappy编解码器在此阶段压缩数据|
|mapreduce.output.fileoutputformat.compress(在mapred-site.xml中配置)|false|reducer输出|这个参数设为`true`启用压缩|
|mapreduce.output.fileoutputformat.compress.codec(在mapred-site.xml中配置)|org.apache.hadoop.io.compress.DefaultCodec|reducer输出|使用标准工具或者编解码器,如gzip和bzip2|



## 案例使用


1. 开启MapTask的输出压缩
   在job上加入
```
//开启map端输出压缩
conf.setBoolean("mapreduce.map.output.compress",true);

//设置map端输出压缩方式  参数:压缩方式,编码方式
conf.setClass("mapreduce.map.output.compress.codec",BZipCodec.claas,CompressionCodec.claas);
```

2.开启reduceTask的输出压缩
在job上加入
```
//设置reduce端输出压缩开启
FileOutputFormat.setCompressOutput(job.true);

//设置支持压缩的方式 选择一个
FileOutputFormat.setOutputCompressorClass(jon,BZipCodec.class);
//FileOutputFormat.setOutputCompressorClass(jon,GzipCodec.class);
//FileOutputFormat.setOutputCompressorClass(jon,DefaultCodec.class);
```