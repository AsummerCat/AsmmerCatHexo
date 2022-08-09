---
title: log4j日志模板输出
date: 2022-02-06 21:49:44
tags: [java]
---

# log4j日志模板输出

## 配置文件
```
logging.config=classpath:log4j2.properties
```

<!--more-->

## 正文
```
status = error
monitorInterval=30

# 日志存放路径
property.LOG_HOME=/目录/org-organization/logs/
# 日志名称
property.SERVER_NAME=org-organization
# 日志切片大小
property.FILE_SIZE=50M
property.FILE_MAX=20
property.LOG_LEVEL=INFO
property.LOG_PATTERN=[${SERVER_NAME}] [%d{yyyy-MM-dd HH:mm:ss.SSS}] [%t] %-5p => %c.%M(%F:%L) - %m%n

appender.console.type=Console
appender.console.name=ConsoleAppender
appender.console.layout.type=PatternLayout
appender.console.layout.pattern=${LOG_PATTERN}

appender.rolling.type=RollingFile
appender.rolling.name=FileAppender
appender.rolling.filter.threshold.level=trace
appender.rolling.filter.threshold.type=ThresholdFilter
appender.rolling.fileName=${LOG_HOME}/${SERVER_NAME}.log
appender.rolling.filePattern=${LOG_HOME}/${SERVER_NAME}-%d{yyyy-MM-dd}-%i.log
appender.rolling.layout.type=PatternLayout
appender.rolling.layout.pattern=${LOG_PATTERN}
appender.rolling.policies.type=Policies
appender.rolling.policies.time.type=TimeBasedTriggeringPolicy
appender.rolling.policies.time.interval=2
appender.rolling.policies.time.modulate=true
appender.rolling.policies.size.type=SizeBasedTriggeringPolicy
appender.rolling.policies.size.size=${FILE_SIZE}
appender.rolling.strategy.type = DefaultRolloverStrategy
appender.rolling.strategy.max=${FILE_MAX}
appender.rolling.strategy.delete.type=Delete
appender.rolling.strategy.delete.basePath=${LOG_HOME}
appender.rolling.strategy.delete.maxDepth=1
appender.rolling.strategy.delete.iffile.type=IfFileName
appender.rolling.strategy.delete.iffile.glob=${SERVER_NAME}-*.log
appender.rolling.strategy.delete.iflastmodify.type=IfLastModified
appender.rolling.strategy.delete.iflastmodify.age=7d

rootLogger.level=${LOG_LEVEL}
rootLogger.appenderRef.console.ref=ConsoleAppender
rootLogger.appenderRef.file.ref=FileAppender

logger.archetype.name=com.xxx.xxx
logger.archetype.level=${LOG_LEVEL}
logger.archetype.additivity=false
logger.archetype.appenderRef.console.ref=ConsoleAppender
logger.archetype.appenderRef.file.ref=FileAppender

logger.sql.name=com.syswin.systoon
logger.sql.level=TRACE
logger.sql.additivity=false
logger.sql.appenderRef.console.ref=ConsoleAppender
logger.sql.appenderRef.file.ref=FileAppender

logger.mybatis.name=org.apache.ibatis
logger.mybatis.level=DEBUG
logger.mybatis.additivity=false
logger.mybatis.appenderRef.console.ref=ConsoleAppender
logger.mybatis.appenderRef.file.ref=FileAppender
```
