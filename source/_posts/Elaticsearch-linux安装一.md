---
title: Elaticsearch-linux安装一
date: 2020-02-14 21:42:45
tags: [linux,Elaticsearch,SpringBoot]
---

# 安装Elaticsearch

需要保证有java的环境



从5.0开始，ElasticSearch 安全级别提高了，不允许采用root帐号启动

## 创建用户 并且赋予操作权限

```
1.创建用户组    
  | groupadd elasticsearch
2.创建用户      创建用户 es 并设置密码为linjing9999
  |useradd es
  |passwd  es
3.用户es 添加到 elasticsearch 用户组
 |usermod -G elasticsearch es

4.将elasticsearch文件权限 赋予elasticsearch组
   chown -R es:elasticsearch /home
5.切换到es账户
  |su es
```

<!--more-->

## 安装ElasticSearch

```
get https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-5.5.2.tar.gz

```

## 访问 http://ip:9200/ 

访问显示

```java
{
  "name" : "h8v_JWT",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "4tyZlslKQx2t2HxCEuClYw",
  "version" : {
    "number" : "6.5.1",
    "build_flavor" : "default",
    "build_type" : "tar",
    "build_hash" : "8c58350",
    "build_date" : "2018-11-16T02:22:42.182257Z",
    "build_snapshot" : false,
    "lucene_version" : "7.5.0",
    "minimum_wire_compatibility_version" : "5.6.0",
    "minimum_index_compatibility_version" : "5.0.0"
  },
  "tagline" : "You Know, for Search"
}
```

表示安装完毕

## 启动参数

```
 ./elasticsearch –d #在后台运行Elasticsearch
 ./elasticsearch -d -Xmx2g -Xms2g #后台启动，启动时指定内存大小（2G）
 ./elasticsearch -d -Des.logger.level=DEBUG  #可以在日志中打印出更加详细的信息。
```



## 测试 -> 开启外网访问

```
vi /conf/elasticsearch.yml
修改network.host: 0.0.0.0
```

## 修改配置文件

```
$ vi config/elasticsearch.yml

#cluster name
cluster.name: sojson-application
#节点名称
node.name: node-1
#绑定IP和端口
network.host: 123.88.88.88
http.port: 9200
# 增加参数，使head插件可以访问es  
http.cors.enabled: true  
http.cors.allow-origin: "*"
```

### 需要注意一点 阿里云默认的内存是不够开启es的

```
因为阿里云系统内存只有2g，所以我们要修改es的config目录下的jvm.options文件
把-Xms1g -Xmx1g改为 -Xms500m -Xmx500m
```

# 安装可视化插件

```
因为elasticsearch-head-master依赖Node环境，所以还要安装Node

安装可视化插件
wget  https://github.com/mobz/elasticsearch-head/archive/master.zip

安装grunt
grunt是基于Node.js的项目构建工具，可以进行打包压缩、测试、执行等等的工作，head插件就是通过grunt启动

##进入到插件目录下面
cd /opt/elasticsearch-6.5.4/es-head/elasticsearch-head-master
##下载安装grunt
npm install -g grunt --registry=https://registry.npm.taobao.org
##检测是否安装成功，如果执行命令后出现版本号就表明成功
grunt -version
##修改源码
Gruntfile.js,添加host正则匹配项
                connect: {
                        server: {
                                options: {
                                        port: 9300,
                                        base: '.',
                                        keepalive: true,
                                        host: '*'
                                }
                        }
                }
_site/app.js，修改es的链接地址
        var ui = app.ns("ui");
        var services = app.ns("services");

        app.App = ui.AbstractWidget.extend({
                defaults: {
                        base_uri: null
                },
                init: function(parent) {
                        this._super();
                        this.prefs = services.Preferences.instance();
                        this.base_uri = this.config.base_uri || this.prefs.get("app-base_uri") || "http://192.168.10.143:9200";
##安装npm的服务，然后再head目录下面启动插件
npm install
grunt server

然后就可以在浏览器中访问ip：9300查看es的结构了。
```



# 安装中文分词器

```
1.下载中文/拼音分词器
IK中文分词器：https://github.com/medcl/elasticsearch-analysis-ik
拼音分词器：https://github.com/medcl/elasticsearch-analysis-pinyin
一定和ES的版本一致（ 6.3.1)

2.进入elasticsearch安装目录/plugins；创建一个mkdir ik
cp 刚才打包的zip文件到ik目录；unzip解压
部署后，记得重启es节点

3.启动时显示 [h8v_JWT] loaded plugin [analysis-ik] 表示安装完成

需要注意的是每个mapping中的需要的字段要设置分词类型 方便检索

ik插件提供了两种分词模式：ik_max_word和ik_smart：
1.ik_max_word：会将文本做最细粒度的拆分，比如会将“中华人民共和国国歌”拆分为“中华人民共和国，中华人民，中华，华人，人民共和国，人民，人民，共和国，共和，和，国国，国歌”，会穷尽各种可能的组合;
2.ik_smart：会做最粗粒度的拆分，比如会将“中华人民共和国国歌”拆分为“中华人民共和国，国歌”。
```



# 启动错误处理

### max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]

```
1.临时提高了vm.max_map_count的大小
 root操作: sysctl -w vm.max_map_count=262144
 查看结果:  sysctl -a|grep vm.max_map_count
 
 2.永久性修改
   cat /etc/sysctl.conf | grep -v "vm.max_map_count" > /tmp/system_sysctl.conf
   echo "vm.max_map_count=262144" >> /tmp/system_sysctl.conf
   mv /tmp/system_sysctl.conf /etc/sysctl.conf
   
```

## initial heap size [268435456] not equal to maximum heap size [536870912]; this can cause resize pauses and prevents mlockall from locking the entire heap

```
jvm内存分配的问题

vim /usr/java/elasticsearch/config/jvm.options
修改 最小堆和最大堆要相等
-Xms512m
-Xmx512m
```

## max file descriptors [65535] for elasticsearch process is too low, increase to at least [65536]

```
vi /etc/security/limits.conf 新增
 es soft nofile 65536
 es hard nofile 65536

用户名+后面的内容

修改后重新登录es用户，使用如下命令查看是否修改成功
ulimit -Hn
```

