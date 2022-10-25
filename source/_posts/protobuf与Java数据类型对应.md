---
title: protobuf与Java数据类型对应
date: 2022-10-25 16:24:43
tags: [protobuf]
---
# protobuf 与 Java 数据类型对应

## 字段规则
required ： 字段只能也必须出现 1 次，多用于必填项，必须赋值的字符
```
required  int32 id = 1 [default = 123]
```
optional ： 字段可出现 0 次或多次，可有可无的字段，可以使用[default = xxx]配置默认值
```
optional string name = 1 [default = "张三"]
```
<!--more-->
repeated ： 字段可出现任意多次（包括 0），多用于 Java List 属性
```
//list String
repeated string strList = 5;
//list 对象
repeated Role roleList = 6;
```

##  字段编号（标识符）
1 ~ 536870911（除去 19000 到 19999 之间的标识号， Protobuf 协议实现中对这些进行了预留。如果非要在.proto 文件中使用这些预留标识号，编译时就会报警）

在消息定义中，每个字段都有唯一的一个标识符。这些标识符是用来在消息的二进制格式中识别各个字段的，一旦开始使用就不能够再改 变。注：[1,15]之内的标识号在编码的时候会占用一个字节。[16,2047]之内的标识号则占用2个字节。所以应该尽可能为那些频繁出现的消息元素保留 [1,15]之内的标识号

#### 系统默认值：

string默认为空字符串
bool默认为false
数值默认为0
enum默认为第一个元素

## 复杂类型

#### Java String、Integer List 在 protobuf 的定义
```
//创建一个 User 对象
message User{
	//list Int
	repeated int32 intList = 1;
	//list String
	repeated string strList = 5;
}
```
#### Java 对象 List 在 protobuf 的定义
```
//创建一个 User 对象
message User{
	//list 对象
	repeated Role roleList = 6;
}
```

####  Map 在 protobuf 的定义
```
//创建一个 User 对象
message User{
	// 定义简单的 Map string
	map<string, int32> intMap = 7;
	// 定义复杂的 Map 对象
	map<string, string> stringMap = 8;
}
```

#### Java 对象 Map 在 protobuf 的定义
```
//创建一个 User 对象
message User{
	// 定义复杂的 Map 对象
	map<string, MapVauleObject> mapObject = 8;
}


// 定义 Map 的 value 对象
message MapVauleObject {
	string code = 1;
	string name = 2;
}
```

#### Java 实体类中嵌套实体 在 protobuf 的定义
```
//创建一个 User 对象
message User{
	// 对象
	NickName nickName = 4;
}

// 定义一个新的Name对象
message NickName {
	string nickName = 1;
}
```

## protobuf 和 JSON 互相转换
使用这个转换必须要使用 protobuf 的 java util jar 包
```
<!--  proto 与 Json 互转会用到-->
<dependency>
    <groupId>com.google.protobuf</groupId>
    <artifactId>protobuf-java-util</artifactId>
    <version>3.15.3</version>
</dependency>
```
#### protobuf 转 Json
```
String json = JsonFormat.printer().print(sourceMessage);
```

#### Json 转 protobuf
```
//ignoringUnknownFields 如果 json 串中存在的属性 proto 对象中不存在，则进行忽略，否则会抛出异常
JsonFormat.parser().ignoringUnknownFields().merge(json, targetBuilder);
return targetBuilder.build();
```

#### protobuf 和 JSON 互转工具类
```
package com.wxw.notes.protobuf.util;

import com.google.protobuf.InvalidProtocolBufferException;
import com.google.protobuf.Message;
import com.google.protobuf.util.JsonFormat;

/**
 * <ul> 注意：
 *  <li>该实现无法处理含有Any类型字段的Message</li>
 *  <li>enum类型数据会转化为enum的字符串名</li>
 *  <li>bytes会转化为utf8编码的字符串</li>
 * </ul> 以上这段暂未进行测试
 *
 * @author wuxiongwei
 * @date 2021/5/13 16:04
 * @Description proto 与 Json 转换工具类
 */
public class ProtoJsonUtil {
    /**
     * proto 对象转 JSON
     * 使用方法： //反序列化之后
     *             UserProto.User user1 = UserProto.User.parseFrom(user);
     *             //转 json
     *             String jsonObject = ProtoJsonUtil.toJson(user1);
     * @param sourceMessage proto 对象
     * @return 返回 JSON 数据
     * @throws InvalidProtocolBufferException
     */
    public static String toJson(Message sourceMessage) throws InvalidProtocolBufferException {
        if (sourceMessage != null) {
            String json = JsonFormat.printer().includingDefaultValueFields().print(sourceMessage);
            return json;
        }
        return null;
    }

    /**
     * JSON 转 proto 对象
     * 使用方法：Message message = ProtoJsonUtil.toObject(UserProto.User.newBuilder(), jsonObject);
     * @param targetBuilder proto 对象 bulider
     * @param json          json 数据
     * @return 返回转换后的 proto 对象
     * @throws InvalidProtocolBufferException
     */
    public static Message toObject(Message.Builder targetBuilder, String json) throws InvalidProtocolBufferException {
        if (json != null) {
            //ignoringUnknownFields 如果 json 串中存在的属性 proto 对象中不存在，则进行忽略，否则会抛出异常
            JsonFormat.parser().ignoringUnknownFields().merge(json, targetBuilder);
            return targetBuilder.build();
        }
        return null;
    }
}
```