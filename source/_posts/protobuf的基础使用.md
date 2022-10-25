---
title: protobuf的基础使用
date: 2022-10-25 16:24:28
tags: [protobuf]
---
# protobuf的基础使用

## 需要下载protobuf的程序来构建
```
https://github.com/protocolbuffers/protobuf/releases
protoc-21.8-win64.zip

构建环境变量 path中加入
E:\protoc-21.8-win64\bin

验证
protoc
protoc --version
```

## 1.idea 安装 protobuf 相关插件
```
一个是根据 .proto 文件来生成 proto 对象

一个是使得 idea 支持我们的 proto 语法，例如关键词高亮等功能
```
```
protobuf Generator
protobuf Support
```
<!--more-->

## 2.检查是否安装成功
重启之后我们可以在工具栏(tool)看到这两个选项

一个是配置全局的 protobuf

一个是生成所有的 protobuf 文件

## 3.配置全局 protobuf
```
protoc path ：我们下载的 protobuf 编辑器的位置，在 bin 目录下有一个 .exe 文件

quick gen : 对应的语言， Java
```

## 4.protobuf文件定义 proto3 对应各语言类型



| .proto | Type	Notes | C++ Type | Java Type|Python Type| Go Type |
| --- | --- | --- | --- | --- | --- |
| double | |double | double | float | *float64 |
|float||float|float|float|*float32|
|int32|使用可变长度编码。编码负数的效率低 - 如果你的字段可能有负值，请改用 sint32|int32|int|int|*int32|
|int64|使用可变长度编码。编码负数的效率低 - 如果你的字段可能有负值，请改用 sint64	|int64|long|int/long[3]|*int64|
|uint32|使用可变长度编码|uint32|int[1]|int/long[3]|*uint32|
|uint64|使用可变长度编码|uint64|long[1]|int/long[3]|*uint64|
|sint32|使用可变长度编码。有符号的 int 值。这些比常规 int32 对负数能更有效地编码|int32|int|int|*int32|
|sint64|使用可变长度编码。有符号的 int 值。这些比常规 int64 对负数能更有效地编码|int64|long|int/long[3]|*int64|
|fixed32|总是四个字节。如果值通常大于 228，则比 uint32 更有效。|uint32|int[1]|int/long[3]|*uint32|
|fixed64|总是八个字节。如果值通常大于 256，则比 uint64 更有效。|uint64|long[1]|int/long[3]|*uint64|
|sfixed32|总是四个字节|int32|int|int|*int32|
|sfixed64|总是八个字节|int64|long|int/long[3]|*int64|
|bool||bool|boolean|bool|*bool|
|string|字符串必须始终包含 UTF-8 编码或 7 位 ASCII 文本|string|String|str/unicode[4]|*string|
|bytes|可以包含任意字节序列|string|ByteString|str|[]byte|

## 5.引入依赖
引入相关依赖，这里的依赖版本和我们的编辑器一个版本就好
```
        <!--  protobuf 支持 Java 核心包-->
        <dependency>
            <groupId>com.google.protobuf</groupId>
            <artifactId>protobuf-java</artifactId>
            <version>3.21.8</version>
        </dependency>


        <!--  proto 与 Json 互转会用到-->
        <dependency>
            <groupId>com.google.protobuf</groupId>
            <artifactId>protobuf-java-util</artifactId>
            <version>3.21.8</version>
        </dependency>
```
## 6.编写 .proto 文件
在 resource 资源文件夹下面创建一个 proto 文件夹

新建一个 xxxx.proto

内容如下
```
syntax = "proto3";
//使用 proto3 语法 ,未指定则使用proto2 注意第一行不允许注释

//生成 proto 文件所在包路径
package com.wxw.notes.protobuf.proto;

//生成 proto 文件所在包路径
option java_package = "com.wxw.notes.protobuf.proto";

//生成 proto 文件名
option java_outer_classname="DemoProto";

message Demo{
  //自身属性
  int32 id = 1;
  string code = 2;
  string name = 3;
}
```
## 7.生成 proto 对象
选中我们新建的.proto 文件，右键，选择框中的`quick gen protobuf hero`选项就可以生成了
会生成一个对应`java_outer_classname`的java文件
```
里面包含构建方法 和一个Builder构造函数
```

## 8.protobuf 序列化和反序列化
```
package com.linjingc.protobufdemo.protobuf;

import com.google.protobuf.InvalidProtocolBufferException;
import com.google.protobuf.util.JsonFormat;

import java.util.Arrays;

public class sys {

    public static void test(){
    //初始化数据
    DemoProto.Demo.Builder demo = DemoProto.Demo.newBuilder();
        demo.setId(1)
                .setCode("001")
                .setName("张三")
                .build();

    //序列化
    DemoProto.Demo build = demo.build();
    //转换成字节数组
    byte[] s = build.toByteArray();
        System.out.println("protobuf数据bytes[]:" + Arrays.toString(s));
        System.out.println("protobuf序列化大小: " + s.length);


    DemoProto.Demo demo1 = null;
    String jsonObject = null;
        try {
        //反序列化
        demo1 = DemoProto.Demo.parseFrom(s);
        //转 json
        jsonObject = JsonFormat.printer().print(demo1);

    } catch (
    InvalidProtocolBufferException e) {
        e.printStackTrace();
    }

        System.out.println("Json格式化结果:\n" + jsonObject);
        System.out.println("Json格式化数据大小: " + jsonObject.getBytes().length);
}

    public static void main(String[] args) {
        test();
    }
}

```

## 