---
title: netty入门-基础搭建一
date: 2019-11-12 13:59:20
tags: [netty]
---

# netty入门-基础搭建一

[demo地址](https://github.com/AsummerCat/netty-demo)

[参考地址](https://www.cnblogs.com/xujian2014/p/5704316.html)

## 介绍

- API 使用简单，开发门槛低；
- 定制能力强，可以通过 ChannelHandler 对通信框架进行灵活的扩展；
- 性能高，通过与其它业界主流的 NIO 框架对比，Netty 的综合性能最优；

## 导入pom

```jav
<dependency>
    <groupId>io.netty</groupId>
    <artifactId>netty-all</artifactId>
    <version>4.1.43.Final</version>
</dependency>
```

<!--more-->

## 服务端

```
/**
	 * 连接客户端
	 */
	public static ConcurrentHashMap<String, ChannelHandlerContext> map = new ConcurrentHashMap<String, ChannelHandlerContext>();
	/**
	 * 维护连接上的客户端
	 */
	private static ChannelGroup channelGroup = new DefaultChannelGroup(GlobalEventExecutor.INSTANCE);
	private static ChannelFuture serverChannelFuture;



	public static void main(String[] args) {

		ServerBootstrap serverBootstrap = new ServerBootstrap();

		//接收客户端连接
		NioEventLoopGroup boos = new NioEventLoopGroup();
		//处理已连接客户端请求
		NioEventLoopGroup worker = new NioEventLoopGroup();
		try {
			//分组 绑定线程池
			serverBootstrap.group(boos, worker);
			//管道
			serverBootstrap.channel(NioServerSocketChannel.class);
			// 2小时无数据激活心跳机制
			serverBootstrap.option(ChannelOption.SO_KEEPALIVE, true);
			//指定此套接口排队的最大连接个数
			serverBootstrap.option(ChannelOption.SO_BACKLOG, 1024);
			//来监听已经连接的客户端的Channel的动作和状态。
			serverBootstrap.childHandler(new ChannelInitializer<NioSocketChannel>() {

				                             @Override
				                             protected void initChannel(NioSocketChannel nioSocketChannel) throws Exception {
					             /*
					             LineBasedFrameDecoder的工作原理是：依次遍历ByteBuf中的可读字节，
                                判断看其是否有”\n” 或 “\r\n”， 如果有就以此位置为结束位置。
                                从可读索引到结束位置的区间的字节就组成了一行。 它是以换行符为结束标志的解码器，
                                支持携带结束符和不带结束符两种解码方式，同时支持配置单行的最大长度，
                                如果读到了最大长度之后仍然没有发现换行符，则抛出异常，同时忽略掉之前读到的异常码流
					              */
					                             nioSocketChannel.pipeline().addLast(new LineBasedFrameDecoder(1024));

					                             //字符串解码和编码
					                             //LineBasedFrameDecoder + StringDecoder 就是一个按行切换的文本解码器。
					                             nioSocketChannel.pipeline().addLast(new StringDecoder());
					                             nioSocketChannel.pipeline().addLast(new StringEncoder());

									            //发送消息频率。单位秒。此设置是60秒发送一次消息
					                             //readerIdleTime为读超时时间（即测试端一定时间内未接受到被测试端消息）
					                             //writerIdleTime为写超时时间（即测试端一定时间内向被测试端发送消息）
					                             //allIdleTime：所有类型的超时时间
					                             nioSocketChannel.pipeline().addLast(new IdleStateHandler(60, 60, 60, TimeUnit.SECONDS));

					                             //接受客户端消息
					                             nioSocketChannel.pipeline().addLast(new SimpleChannelInboundHandler<String>() {
						                             /**
						                              * 读取客户端消息
						                              */
						                             @Override
						                             protected void channelRead0(ChannelHandlerContext channelHandlerContext, String msg) throws Exception {
                                                         System.out.println(msg);
							                             map.put(msg, channelHandlerContext);
							                             //回复
							                             NettyServer.sendMessageClient(msg);
						                             }

						                             /**
						                              * 服务端监听到客户端活动
						                              */
						                             @Override
						                             public void channelActive(ChannelHandlerContext ctx) throws Exception {
							                             //移除全局用户中的这个人
							                             channelGroup.add(ctx.channel());
							                             System.out.println(ctx.channel().localAddress().toString()+"已经成功连接");
							                             //发送全体广播
							                             sendAllMessage();
						                             }

						                             /**
						                              * 服务端监听到客户端不活动
						                              */
						                             @Override
						                             public void channelInactive(ChannelHandlerContext ctx) throws Exception {
							                             //移除全局用户中的这个人
							                             channelGroup.remove(ctx.channel());
							                             System.out.println(ctx.channel().localAddress().toString()+"已经断开");
						                             }
					                             });
				                             }
			                             }
			);
			//启动netty服务
			//serverBootstrap.bind(8000);
			serverChannelFuture= serverBootstrap.bind(8000).sync();
		}catch (Exception e){
			// 释放线程池资源
			boos.shutdownGracefully();
			worker.shutdownGracefully();
			e.printStackTrace();
		}
	}
```

## 需要注意的内容

```
如果在Handler中

开启了 nioSocketChannel.pipeline().addLast(new LineBasedFrameDecoder(1024));
服务端接收消息 必须带上回车换行   \n

客户端也是如果开启了这个 也需要带上回车换行

```

## 心跳监听

```


nioSocketChannel.pipeline().addLast(new IdleStateHandler(60, 60, 60, TimeUnit.SECONDS));
前三个的参数解释如下：

1）readerIdleTime：为读超时时间（即测试端一定时间内未接受到被测试端消息）

2）writerIdleTime：为写超时时间（即测试端一定时间内向被测试端发送消息）

3）allIdleTime：所有类型的超时时间

```



## 群体广播

```
	/**
	 * 群发广播
	 */
	public static void sendAllMessage(){
		channelGroup.writeAndFlush("新用户登录了"+new Date());
	}
```



## 发送消息给客户端

```
/**
	 * 发送消息给客户端
	 */
	public static void sendMessageClient(String msg) {
		//*****回复*********
		//1.判断客户端是否在线
		ChannelHandlerContext client = map.get(msg);
		if (client == null) {
			return;
		}
		//2.回复客户端
		if (!client.channel().isActive()) {
			System.out.println("客户端下线");
		}
		client.channel().writeAndFlush("嘿嘿 你来了");
		//*****回复*********
	}
```



## 服务端的NioEventLoopGroup

1. `NioEventLoopGroup` 是用来处理I/O操作的多线程事件循环器，Netty 提供了许多不同的 `EventLoopGroup`的实现用来处理不同的传输。在这个例子中我们实现了一个服务端的应用，因此会有2个 NioEventLoopGroup 会被使用。第一个经常被叫做‘boss’，用来接收进来的连接。第二个经常被叫做‘worker’，用来处理已经被接收的连接，一旦‘boss’接收到连接，就会把连接信息注册到‘worker’上。如何知道多少个线程已经被使用，如何映射到已经创建的 `Channel`上都需要依赖于 EventLoopGroup 的实现，并且可以通过构造函数来配置他们的关系。
2. `ServerBootstrap`是一个启动 NIO 服务的辅助启动类。你可以在这个服务中直接使用 Channel，但是这会是一个复杂的处理过程，在很多情况下你并不需要这样做。
3. 这里我们指定使用 `NioServerSocketChannel` 类来举例说明一个新的 Channel 如何接收进来的连接。
4. 这里的事件处理类经常会被用来处理一个最近的已经接收的 Channel。SimpleChatServerInitializer 继承自`ChannelInitializer`是一个特殊的处理类，他的目的是帮助使用者配置一个新的 Channel。也许你想通过增加一些处理类比如 SimpleChatServerHandler 来配置一个新的 Channel 或者其对应的`ChannelPipeline` 来实现你的网络程序。当你的程序变的复杂时，可能你会增加更多的处理类到 pipline 上，然后提取这些匿名类到最顶层的类上。
5. 你可以设置这里指定的 Channel 实现的配置参数。我们正在写一个TCP/IP 的服务端，因此我们被允许设置 socket 的参数选项比如tcpNoDelay 和 keepAlive。请参考 `ChannelOption` 和详细的 `ChannelConfig`实现的接口文档以此可以对ChannelOption 的有一个大概的认识。
6. option() 是提供给`NioServerSocketChannel` 用来接收进来的连接。childOption() 是提供给由父管道 `ServerChannel`接收到的连接，在这个例子中也是 NioServerSocketChannel。
7. 我们继续，剩下的就是绑定端口然后启动服务。这里我们在机器上绑定了机器所有网卡上的 8080 端口。当然现在你可以多次调用 bind() 方法(基于不同绑定地址)。

## 服务端的ChannelInboundHandler

详细内容

1. `SimpleChatServerHandler` 继承自 `SimpleChannelInboundHandler`，这个类实现了 `ChannelInboundHandler`接口，`ChannelInboundHandler `提供了许多事件处理的接口方法，然后你可以覆盖这些方法。现在仅仅只需要继承 `SimpleChannelInboundHandler` 类而不是你自己去实现接口方法。
2. 覆盖了 `handlerAdded()` 事件处理方法。每当从服务端收到新的客户端连接时，客户端的 Channel 存入 ChannelGroup列表中，并通知列表中的其他客户端 Channel
3. 覆盖了 `handlerRemoved() `事件处理方法。每当从服务端收到客户端断开时，客户端的 Channel 自动从 ChannelGroup 列表中移除了，并通知列表中的其他客户端 Channel
4. 覆盖了 `channelRead0() `事件处理方法。每当从服务端读到客户端写入信息时，将信息转发给其他客户端的 Channel。其中如果你使用的是 Netty 5.x 版本时，需要把 channelRead0() 重命名为messageReceived()
5. 覆盖了 `channelActive() `事件处理方法。服务端监听到客户端活动
6. 覆盖了 `channelInactive()` 事件处理方法。服务端监听到客户端不活动
7. `exceptionCaught() `事件处理方法是当出现 Throwable 对象才会被调用，即当 Netty 由于 IO 错误或者处理器在处理事件时抛出的异常时。在大部分情况下，捕获的异常应该被记录下来并且把关联的 channel 给关闭掉。然而这个方法的处理方式会在遇到不同异常的情况下有不同的实现，比如你可能想在关闭连接之前发送一个错误码的响应消息。

## 简单使用介绍

```
> netty使用很简单，只要加入相应处理handle即可。
>
> 传输层为了效率，tcp协议发送数据包时有可能合并发送，对接收方来说会产生粘包问题，需要在应用层解决拆包，收发数据时协商设计分割点，一般而言有四种分割收到包的方法：
>
> 1. 发送方在发送每段数据后拼接回车换行符，接收方读到“\r\n”则认为是一个独立的数据包。netty默认解析实现是LineBasedFrameDecoder，加入解码handle即可。
> 2. 其他自定义分割符号，如“#”。netty实现handle是DelimiterBasedFrameDecoder.
> 3. 无论数据大小，每次发送固定长度，如1024字节，不够的0补位，超出的截断。缺点是比较生硬，数据小的时候浪费带宽资源。netty实现的handle是FixedLengthFrameHandle.
> 4. 数据分为消息头，消息体，消息头定义消息体长度，接收端解析出长度后只读取指定的长度。需要自己实现decoder。

_上述DecoderHandle全部继承ByteToMessageDecoder,是netty封装的解析二进制数据的处理类，只要将相应handle添加到pipeline中即可，解析完成后传输给自定义的逻辑处理类MyServerHandler。此项目中与c端约定传输json字符串格式数据，每段数据手动增加换行分割符。_ 
```



## 客户端

```

public class NettyClient {
	public static void main(String[] args) throws InterruptedException {
		Bootstrap bootstrap = new Bootstrap();
		NioEventLoopGroup client = new NioEventLoopGroup();
		bootstrap
				.group(client)
				.channel(NioSocketChannel.class)
				.handler(new ChannelInitializer<Channel>() {
					@Override
					protected void initChannel(Channel channel) throws Exception {
						//字符串编码解码
						channel.pipeline().addLast("decoder", new StringDecoder());
						channel.pipeline().addLast("encoder", new StringEncoder());
						//心跳检测
						channel.pipeline().addLast(new IdleStateHandler(0, 4, 0, TimeUnit.SECONDS));
						//接受服务端消息
						channel.pipeline().addLast(new SimpleChannelInboundHandler<String>() {
							@Override
							protected void channelRead0(ChannelHandlerContext channelHandlerContext, String msg) throws Exception {
								System.out.println("客户端收到消息了" + msg);
							}


						});
					}
				});

		//客户端连接
		Channel channel = bootstrap.connect("127.0.0.1", 8000).channel();
		while (true) {
			channel.writeAndFlush(new Date() + ":hello world\n");
			Thread.sleep(2000);
		}

	}
```

