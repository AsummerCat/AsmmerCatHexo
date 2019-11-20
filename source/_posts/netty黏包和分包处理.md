---
title: netty黏包和分包处理
date: 2019-11-20 10:31:16
tags: [netty]
---

# netty 自带拆包类

## demo地址

[netty-demo](https://github.com/AsummerCat/netty-demo)

```java
Netty自带拆包类

自己实现拆包虽然可以细粒度控制, 但是也会有些不方便, 可以直接调用Netty提供的一些内置拆包类.

FixedLengthFrameDecoder 按照特定长度组包

DelimiterBasedFrameDecoder 按照指定分隔符组包, 例如本文中的$$$

LineBasedFrameDecoder 按照换行符进行组包, \r \n等等

```

<!--more-->

## LineBasedFrameDecoder

按照换行符进行组包, \r \n等等

### 服务端

```
 nioSocketChannel.pipeline().addLast(new LineBasedFrameDecoder(1024));
```

### 客户端

以`\n`为结尾

```
channel.writeAndFlush(new Date() + ":hello world"+" num"+i+"\n");
```



## DelimiterBasedFrameDecoder

DelimiterBasedFrameDecoder用来解决以特殊符号作为消息结束符的粘包问题

按照指定的字符进行分包

DelimiterBasedFrameDecoder的构造方法：

```
public DelimiterBasedFrameDecoder(int maxFrameLength, boolean stripDelimiter, ByteBuf delimiter) {
        this(maxFrameLength, stripDelimiter, true, delimiter);
}m
axFrameLength：解码的帧的最大长度
stripDelimiter：解码时是否去掉分隔符
failFast：为true，当frame长度超过maxFrameLength时立即报TooLongFrameException异常，为false，读取完整个帧再报异常
delimiter：分隔符
```

### 服务端

```java
   //需要指定需要分包的字符
ByteBuf delimiter = Unpooled.copiedBuffer("$$".getBytes());
nioSocketChannel.pipeline().addLast(new DelimiterBasedFrameDecoder(1024,delimiter));
```



### 客户端

```
	channel.writeAndFlush(new Date() + ":hello world"+" num"+i+"$$");
```



## FixedLengthFrameDecoder

FixedLengthFrameDecoder用来解决定长消息的粘包问题

FixedLengthFrameDecoder是固定长度解码器，它能够按照指定的长度对消息进行自动解码。

### 服务端

```java
 //固定长度解码器
nioSocketChannel.pipeline().addLast(new FixedLengthFrameDecoder(100));//参数为一次接受的数据长度

```



### 客户端

```java
	/**
			 * 生成固定长度byte方式一：使用String 拼接
			 */
			String s1 = new Date() + ":hello world by FixedLengthFrameDecoder"+" num"+i;
			byte[] bytes1 = s1.getBytes("UTF-8");
			byte[] msgBytes1 = new byte[100];
			for (int i1 = 0; i1 < msgBytes1.length; i1++) {
				if (i1 < bytes1.length) {
					msgBytes1[i1] = bytes1[i1];
				} else {
					/**32 表示空格，等价于：msgBytes1[i] = " ".getBytes()[0];*/
					msgBytes1[i1] = 32;
				}
			}

			/**
			 * 生成固定长度byte方式二：使用 System.arraycopy 快速复制数组
			 */
			byte[] bytes2 = s1.getBytes("UTF-8");
			byte[] msgBytes2 = new byte[100];
			System.arraycopy(bytes2, 0, msgBytes2, 0, bytes2.length);
			System.out.println(new String(msgBytes2) + "," + msgBytes2.length);
```



需要注意的是 只要一种就可以了 不然会导致 服务器解析格式错误 乱七八糟的