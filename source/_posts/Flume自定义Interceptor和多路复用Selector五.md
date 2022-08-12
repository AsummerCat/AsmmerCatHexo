---
title: Flume自定义Interceptor和多路复用Selector五
date: 2022-08-12 09:44:00
tags: [Flume,大数据]
---
# Flume自定义Interceptor和多路复用Selector

可以实现 同数据源发送给不同sink不同的消息;

实现过程: 根据 header中的头消息进行区分

# 实现

## 导入pom
```
<!-- flume核心依赖 -->
 <dependency>
     <groupId>org.apache.flume</groupId>
     <artifactId>flume-ng-core</artifactId>
     <version>1.9.0</version>
 </dependency>
```
<!--more-->

## 实现拦截器接口
实现`org.apache.flume.interceptor.Interceptor`
```
package flume;
 
import org.apache.flume.Context;
import org.apache.flume.Event;
import org.apache.flume.interceptor.Interceptor;
 
import java.util.ArrayList;
import java.util.List;
 
public class MyInterceptor implements Interceptor {
 
    @Override
    public void initialize() {
 
    }
 
    /**
     * 拦截器实现方法
     * @param event 一个事件 ，封装一行的数据 ，有header和body（具体的数据）
     * @return
     */
    @Override
    public Event intercept(Event event) {
        //获取数据body
        byte[] oldbody = event.getBody();
        //将数据转换为大写
        byte[] bytes = new String(oldbody).toUpperCase().getBytes();
        //封装-setBody()没有返回值，所以不能在return里面写
        event.setBody(bytes);

//3.根据 body 中是否有"A"来决定添加怎样的头信息
 if (body.contains("A")) {
 //4.添加头信息
 headers.put("type", "first");
 } else {
 //4.添加头信息
 headers.put("type", "second");
 }
        
        // 返回
        return event;
    }
 
    @Override
    public List<Event> intercept(List<Event> list) {
        ArrayList<Event> eventArrayList = new ArrayList<>();
        //循环将每个事件的数据转换成大写
        for(Event event:list){
            eventArrayList.add(intercept(event));
        }
        return eventArrayList;
    }
 
    /**
     * 关闭资源
     */
    @Override
    public void close() {
 
    }
 
    //定义一个内部类
    public static class Builder implements Interceptor.Builder{
 
        @Override
        public Interceptor build() {
            return new MyInterceptor();
        }
 
        @Override
        public void configure(Context context) {
 
        }
    }
```

## 打包做成jar
上传到`flume`下的`lib`目录中

## 创建flume启动配置文件
```
#1 agent
# 设置一个数据源  两个输出源 两个通道
a1.sources = r1
a1.sinks = k1 k2
a1.channels = c1 c2
 
#2 source
a1.sources.r1.type = exec
a1.sources.r1.command = tail -F /opt/plus

#定义拦截器

a1.sources.r1.interceptors = i1

# 全类名$Builder的类名
a1.sources.r1.interceptors.i1.type = 全类名$Builder

# 设置多路复用模式
a1.sources.r1.selector.type = multiplexing
# 头消息根据 type字段区分
a1.sources.r1.selector.header = type
# 区分后的selector
a1.sources.r1.selector.mapping.first = c1
a1.sources.r1.selector.mapping.second = c2


# 设置两个输出源
a1.sinks.k1.type = logger
a1.sinks.k2.type = logger

# 设置两个通道
a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000
a1.channels.c1.transactionCapacity = 100
a1.channels.c2.type = memory
a1.channels.c2.capacity = 1000
a1.channels.c2.transactionCapacity = 100

# 设置一个数据源有两个输出源
a1.sources.r1.channels = c1 c2

#设置输出源对应的管道
a1.sinks.k1.channel = c1
a1.sinks.k1.channel = c2
```