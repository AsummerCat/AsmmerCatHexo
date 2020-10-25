---
title: ElasticSearch笔记ik分词器相关(二十三)
date: 2020-08-23 21:01:05
tags: [elasticSearch笔记]
---

## 中文分词器 ik

要下载对应版本 然后放入`es/plugins/ik`目录下
重启es

### ik分词器基础知识
1.  `ik_max_word` :会将文本做最细粒度的拆分
2.  `ik_smart`: 会做最粗粒度的拆分,比如`中华人民共和国国歌`,会被拆分为`(中华人民共和国)  与 (国歌)`

<!--more-->

### ik分词的使用
```
PUT /my_index
{
    "mappings":{
        "properties":{
            "text":{
                "type":"text",
                "analyzer":"ik_max_word"
            }
        }
    }
}

创建索引指定字段送ik分词器
```

### 配置详解
配置文件地址: `es/plugins/ik/config`目录里的`IKAnalyzer.cfg.xml`

```
'IKAnalyzer.cfg.xml' : 用来配置自定义词库的

'main.dic': ik的原生内置的中文词库 总共27w多条

'quantifier.dic' :单位相关的中文词库

'suffix.dic'  :后缀名相关的中文词库

'surname.dic' : 姓氏的中文词库

'stopword.dic' : 英文的停用词词库

一般 像停用词,会在分词的时候被干掉
不会建立倒排索引中
```



### 自定义词库
主要目的是: 每年的流行词都会新增  
比如: 难受香菇

还有一些就是 专业名称:
比如阿莫西林等等

配置的地方: 
`在IKAnalyzer.cfg.xml的key_dict`

需要重启才能生效


## 动态修改ik分词器词库基于mysql热更新
```
 修改jdbc相关配置
 
    在mysql中添加词库与停用词

```
### 热更新方案
1. 修改ik分词器源码,然后手动支持从mysql中每隔一定时间,自动加载新的词库<font color="red">(推荐)</font>
2. 基于ik分词器原生支持的热更新方案,部署一个web服务器,提供一个http接口,通过modified和tag两个http消息头,来提供词语的热更新

```
修改源码 添加mysql连接
```