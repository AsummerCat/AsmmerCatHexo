---
title: ElasticSearch笔记Docker下安装ElasticSearch和Kibana-(零)
date: 2020-08-11 13:57:55
tags: [ElasticSearch笔记]
---

# ElasticSearch笔记Docker下安装ElasticSearch和Kibana

# ElasticSearch安装
```
docker pull elasticsearch:7.2.0
```
### 启动es
```
docker run --name elasticsearch -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" -d elasticsearch:7.2.0
```
### 测试启动成功
```
curl http://localhost:9200
```
<!--more-->
### 修改配置，解决跨域访问问题
首先进入到容器中，然后进入到指定目录修改elasticsearch.yml文件。
```
docker exec -it elasticsearch /bin/bash
cd /usr/share/elasticsearch/config/
vi elasticsearch.yml
```
在elasticsearch.yml的文件末尾加上:
```
http.cors.enabled: true
http.cors.allow-origin: "*"
```
修改配置后重启容器即可。
```
docker restart elasticsearch
```
### 安装分词器
es自带的分词器对中文分词不是很友好，所以我们下载开源的IK分词器来解决这个问题。首先进入到plugins目录中下载分词器，下载完成后然后解压，再重启es即可。具体步骤如下:
注意：elasticsearch的版本和ik分词器的版本需要保持一致，不然在重启的时候会失败。可以在这查看所有版本，选择合适自己版本的右键复制链接地址即可.[地址](https://github.com/medcl/elasticsearch-analysis-ik/releases)
```
cd /usr/share/elasticsearch/plugins/
elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v7.2.0/elasticsearch-analysis-ik-7.2.0.zip
exit
docker restart elasticsearch 


yum install -y unzip zip
```
然后可以在kibana界面的dev tools中验证是否安装成功；
```
PUT /my_index //创建索引
{
    "settings":{
        "number_of_shards":3, 
        "number_of_replicas": 1 
    }
}

POST my_index/_analyze
{
  "analyzer": "ik_max_word",
  "text": "你好我是东邪Jiafly"
}
```
不添加"analyzer": "ik_max_word",则是每个字分词，可以在下面kibana安装完成以后尝试一下。

# kibana安装
同样适用docker安装kibana命令如下:
```
docker pull kibana:7.2.0
```
### 启动kibana
安装完成以后需要启动kibana容器，使用--link连接到elasticsearch容器，命令如下:
```
docker run --name kibana --link=elasticsearch:test  -p 5601:5601 -d kibana:7.2.0
docker start kibana
```
启动以后可以打开浏览器输入http://localhost:5601就可以打开kibana的界面了。


# es7.0以上版本移除了type
```
PUT /my_index1
{
    "settings":{
        "number_of_shards":3,  
        "number_of_replicas": 1
    },
    "mappings" : {
        "properties":{
            "my_field":{"type":"text"}
      }
    }
   
}
```
7.x 以上版本, 使用默认的_doc作为type, 不需要自定义type名称