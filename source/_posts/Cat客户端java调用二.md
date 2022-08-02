---
title: Cat客户端java调用二
date: 2022-08-02 11:07:15
tags: [链路跟踪,cat]
---
# Cat客户端java调用

## 引入pom
```
<dependency>
    <groupId>com.dianping.cat</groupId>
    <artifactId>cat-client</artifactId>
    <version>${cat.version}</version>
</dependency>
```
直接引入jar包
如果没有使用maven管理依赖，可以直接复制 jar/cat-client-3.0.0.jar 到项目 WEB_INF/lib 路径下。

## 配置文件
<!--more-->
需要在你的项目中创建 `src/main/resources/META-INF/app.properties` 文件, 并添加如下内容:
项目名称(仅支持英文或者数字)
```
app.name={appkey}
```
####  配置/data/appdatas/cat/client.xml ($CAT_HOME/client.xml)
此文件用于配置cat-client连接服务端的地址，以10.1.1.1，10.1.1.2，10.1.1.3三台CAT服务器为例
client_cache.xml是路由缓存文件，若路由出现错误，可以删除client_cache.xml，再重启服务
```
    <?xml version="1.0" encoding="utf-8"?>
    <config mode="client">
        <servers>
            <server ip="10.1.1.1" port="2280" http-port="8080"/>
            <server ip="10.1.1.2" port="2280" http-port="8080"/>
            <server ip="10.1.1.3" port="2280" http-port="8080"/>
        </servers>
    </config>
```

2280是默认的CAT服务端接受数据的端口，不允许修改，http-port是Tomcat启动的端口，默认是8080，建议使用默认端口

## 使用
我们提供了一系列 API 来对 Transaction 进行修改。

addData
setStatus
setDurationStart
setDurationInMillis
setTimestamp
complete
```
Transaction t = Cat.newTransaction("URL", "pageName");

try {
    t.setDurationInMillis(1000);
    t.setTimestamp(System.currentTimeMillis());
    t.setDurationStart(System.currentTimeMillis() - 1000);
    t.addData("content");
    t.setStatus(Transaction.SUCCESS);
} catch (Exception e) {
    t.setStatus(e);
    Cat.logError(e);
} finally {
    t.complete();
}
```


##  Event
Cat.logEvent
记录一个事件。

## Cat.logError
记录一个带有错误堆栈信息的 Error。

Error 是一种特殊的事件，它的 type 取决于传入的 Throwable e.

如果 e 是一个 Error, type 会被设置为 Error。
如果 e 是一个 RuntimeException, type 会被设置为 RuntimeException。
其他情况下，type 会被设置为 Exception。
同时错误堆栈信息会被收集并写入 data 属性中。

```
try {
    1 / 0;
} catch (Throwable e) {
    Cat.logError(e);
}
```
Cat.logErrorWithCategory
尽管 name 默认会被设置为传入的 Throwable e 的类名，你仍然可以使用这个 API 来复写它。
```
Exception e = new Exception("syntax error");
Cat.logErrorWithCategory("custom-category", e);
```
就像 logError 一样，你也可以向错误堆栈顶部添加你自己的错误消息：
```
Cat.logErrorWithCategory("custom-category", "?- X = Y, Y = 2", e);
```

## Metric
记录业务指标的总和或平均值。
```
# Counter
Cat.logMetricForCount("metric.key");
Cat.logMetricForCount("metric.key", 3);

# Duration
Cat.logMetricForDuration("metric.key", 5);
```
我们每秒会聚合 metric。

举例来说，如果你在同一秒调用 count 三次（相同的 name），我们会累加他们的值，并且一次性上报给服务端。

在 duration 的情况下，我们用平均值来取代累加值。