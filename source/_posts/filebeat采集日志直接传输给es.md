---
title: filebeat采集日志直接传输给es
date: 2024-05-04 15:42:59
tags: [elk, filebeat]
---
# 安装Filebeat

注意需要跟es版本匹配 目前是7.12版本

## 多版本安装 <https://www.elastic.co/cn/downloads/beats/filebeat>
<!--more-->
    https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-installation-configuration.html


    sudo ./filebeat -e -c filebeat.yml

# 基础配置

```
filebeat.inputs:
- type: log
  enabled: true
  paths:
    - C:\path\to\your\log\file.log

output.elasticsearch:
  hosts: ["your_elasticsearch_host:9200"]
  index: "your_index_name"
  
```

## 比如指定收集指定日志格式

```
在这个配置中，dissect 处理器将日志行根据指定的分隔符进行解析，并将解析后的字段分别存储为 date、info、method 和 message 四个字段。tokenizer 参数指定了解析的模式，field 参数指定了要解析的字段，这里假设日志的内容在 message 字段中。target_prefix 参数为空，表示解析后的数据将替换原始的日志内容。

filebeat.inputs:
- type: log
  enabled: true
  paths:
    - C:\path\to\your\log\file.log
  processors:
    - dissect:
        tokenizer: "%{date} | %{info} | %{method} | %{message}"
        field: "message"
        target_prefix: ""

output.elasticsearch:
  hosts: ["your_elasticsearch_host:9200"]
  index: "your_index_name"

```

## 读取多个日志

```
filebeat.inputs:
- type: log
  enabled: true
  paths:
    - C:\path\to\your\first\service\log\file.log
  processors:
    - dissect:
        tokenizer: "%{date} %{info} %{method} %{message}"
        field: "message"
        target_prefix: ""
  output.elasticsearch:
    hosts: ["your_elasticsearch_host:9200"]
    index: "first_service_index"

- type: log
  enabled: true
  paths:
    - C:\path\to\your\second\service\log\file.log
  processors:
    - dissect:
        tokenizer: "%{date} %{info} %{method} %{message}"
        field: "message"
        target_prefix: ""
  output.elasticsearch:
    hosts: ["your_elasticsearch_host:9200"]
    index: "second_service_index"

```

## 如果需要制定日志编码格式

```
filebeat.inputs:
- type: log
  enabled: true
  paths:
    - C:\path\to\your\log\file.log
  encoding: utf-8

output.elasticsearch:
  hosts: ["your_elasticsearch_host:9200"]
  index: "your_index_name"

```

加上`  encoding: utf-8`

tokenizer: "%{date} \[%{level}] \[%{thread}] \[%{requestId}] %{logger} \[%{line}] - %{message}"

## win

    下载安装包 下载地址：https://www.elastic.co/downloads/beats/filebeat
    解压到指定目录，无需安装
    打开解压后的目录，打开filebeat.yml进行配置。

    ①：配置 Filebeat prospectors->path 这里的路径是所要收集日志的路径 
    ②：配置 enabled: true 这个配置很重要，只有配置为true之后配置才可生效，否则不起作用。 
    ③：配置Outputs ，这里的Outputs有elasticsearch，logstash。按照配置文件下面的示例配置即可。

    打开刚才解压目录 按住ctrl+shift 并同时鼠标右键打开cmd窗口执行命令 
    .\filebeat -e -c filebeat.yml 就可以启动filebeat

    注册exe到服务
    更改PowerShell（管理员启动）执行脚本权限：
    PS C:\tools\filebeat> set-ExecutionPolicy RemoteSigned

    安装filebeat服务：
    PS C:\tools\filebeat> .\install-service-filebeat.ps1

# 完整配置

     .\filebeat -e -c .\filebeat.yml



    filebeat.inputs:
    - type: log
      enabled: true
      paths:
        - D:\mylog\common\common-info.log
      tail_files: true  # 设置为 true，从文件末尾开始读取日志 
      multiline.pattern: '^\d{4}-\d{2}-\d{2} \d{2}:\d{2}:\d{2}' # 开启多行模式 避免被截断日志
      multiline.negate: true
      multiline.match: after  
      processors:
        - dissect:
            tokenizer: "%{date} [%{level}] [%{thread}] [%{requestId}] %{class} [%{line}] - %{message}"      
            field: "message"
            target_prefix: "logData"
    # 注意7.x版本不支持script方法
        - script:  #将class字段和方法行号拼接城一个字段
            lang: painless
            source: >
              ctx.method = ctx.class + ' [' + ctx.line + ']';        

    output.elasticsearch:
      hosts: ["192.168.1.93:9200"]
      index: "common"


    filebeat.config.modules:
      path: ${path.config}/modules.d/*.yml
      reload.enabled: false
    setup.template.settings:
      index.number_of_shards: 1
      #index.codec: best_compression
      #_source.enabled: false

    # 构建创建es索引的名称
    setup.template.name: "common"
    setup.template.pattern: "common-*"
    setup.ilm.enabled: false

    ## 开启filebeat日志
    logging.level: debug

## 如果是多日志收集 那就是多加-type

```
filebeat.inputs:
- type: log
  enabled: true
  paths:
    - D:\mylog\common\common-info.log
  tail_files: true
  multiline.pattern: '^\d{4}-\d{2}-\d{2} \d{2}:\d{2}:\d{2}'
  multiline.negate: true
  multiline.match: after
  processors:
    - dissect:
        tokenizer: "%{date} [%{level}] [%{thread}] [%{requestId}] %{class} [%{line}] - %{message}"
        field: "message"
        target_prefix: "logData"


- type: log  # 添加一个新的输入源
  enabled: true
  paths:
    - D:\mylog\consumer\consumer-info.log
  tail_files: true
  multiline.pattern: '^\d{4}-\d{2}-\d{2} \d{2}:\d{2}:\d{2}'
  multiline.negate: true
  multiline.match: after
  processors:
    - dissect:
        tokenizer: "%{date} [%{level}] [%{thread}] [%{requestId}] %{class} [%{line}] - %{message}"
        field: "message"
        target_prefix: "logData"

- type: log  # 添加一个新的输入源
  enabled: true
  paths:
    - D:\mylog\quartzs\quartzs-info.log
  tail_files: true
  multiline.pattern: '^\d{4}-\d{2}-\d{2} \d{2}:\d{2}:\d{2}'
  multiline.negate: true
  multiline.match: after
  processors:
    - dissect:
        tokenizer: "%{date} [%{level}] [%{thread}] [%{requestId}] %{class} [%{line}] - %{message}"
        field: "message"
        target_prefix: "logData"

output.elasticsearch:
  hosts: ["192.168.1.93:9200"]
  indices:
    - index: "common"
      when.contains:
        log.file.path: "common-info.log"
    - index: "consumer" 
      when.contains:
        log.file.path: "consumer-info.log"
    - index: "quartzs" 
      when.contains:
        log.file.path: "quartzs-info.log"        

filebeat.config.modules:
  path: ${path.config}/modules.d/*.yml
  reload.enabled: false

setup.template.settings:
  index.number_of_shards: 1

setup.template.name: "common"
setup.template.pattern: "common-*"
setup.ilm.enabled: false

setup.template.name: "consumer"
setup.template.pattern: "consumer-*"
setup.ilm.enabled: false

setup.template.name: "quartzs"
setup.template.pattern: "quartzs-*"
setup.ilm.enabled: false

# 开启Filebeat日志
#logging.level: debug

```

