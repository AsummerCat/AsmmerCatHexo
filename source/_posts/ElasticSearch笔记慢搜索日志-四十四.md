---
title: ElasticSearch笔记慢搜索日志(四十四)
date: 2020-09-10 21:51:39
tags: [elasticSearch笔记]
---

跟sql慢查询日志一个道理的

# 设置参数
`elasticsearch.yml`
```
index.search.slowlog.threshold.query.warn: 10s
index.search.slowlog.threshold.query.info: 10s
index.search.slowlog.threshold.query.debug: 2s
index.search.slowlog.threshold.query.trace: 500ms


index.search.slowlog.threshold.fetch.warn: 10s
index.search.slowlog.threshold.fetch.info: 10s
index.search.slowlog.threshold.fetch.debug: 2s
index.search.slowlog.threshold.fetch.trace: 500ms

```


<!--more--> 

## 慢查询日志的日志格式
都是在`log4j2.properties`中配置的
```
appender.index_search_slowlog_rolling.type= rollingFile
appender.index_search_slowlog_rolling.name= index_search_slowlog_rolling

等等....


```


# 索引慢写入日志
```
index.search.slowlog.threshold.index.warn: 10s
index.search.slowlog.threshold.index.info: 10s
index.search.slowlog.threshold.index.debug: 2s
index.search.slowlog.threshold.index.trace: 500ms

```

## 慢写入日志的日志格式
```
appender.index_indexing_slowlog_rolling.type= rollingFile
appender.index_indexing_slowlog_rolling.name= index_search_slowlog_rolling
```