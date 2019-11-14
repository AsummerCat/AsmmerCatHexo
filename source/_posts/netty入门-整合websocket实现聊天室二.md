---
title: netty入门-整合websocket实现聊天室二
date: 2019-11-12 17:36:35
tags: [netty]
---

# netty入门-整合websocket实现聊天室二

[demo地址](https://github.com/AsummerCat/netty-demo)

## 导入pom

```jav
<dependency>
    <groupId>io.netty</groupId>
    <artifactId>netty-all</artifactId>
    <version>4.1.43.Final</version>
</dependency>

  <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-websocket</artifactId>
        </dependency>
```

<!--more-->

# 实现流程

客户端发起握手请求 服务端接收 

然后 就不走http请求 直接创建socket连接  

只有第一次走http请求

## 服务端

## NioWebSocketHandler 初始化Netty服务器HTTP请求处理器和webSocket请求

```java

@Component()
@Qualifier("nioHttpAndWebSocketHandler")
@ChannelHandler.Sharable
public class NioHttpAndWebSocketHandler extends SimpleChannelInboundHandler<Object> {

	private WebSocketServerHandshaker handshaker;
	@Override
	protected void channelRead0(ChannelHandlerContext ctx, Object msg) throws Exception {
		System.out.println("收到消息：" + msg);
		if (msg instanceof FullHttpRequest) {
			//以http请求形式接入，但是走的是websocket
			handleHttpRequest(ctx, (FullHttpRequest) msg);
		} else if (msg instanceof WebSocketFrame) {
			//处理websocket客户端的消息
			handlerWebSocketFrame(ctx, (WebSocketFrame) msg);
		}
	}

	@Override
	public void channelActive(ChannelHandlerContext ctx) throws Exception {
		//添加连接
		System.out.println("客户端加入连接：" + ctx.channel());
		ChannelSupervise.addChannel(ctx.channel());
	}

	@Override
	public void channelInactive(ChannelHandlerContext ctx) throws Exception {
		//断开连接
		System.out.println("客户端断开连接：" + ctx.channel());
		ChannelSupervise.removeChannel(ctx.channel());
	}

	@Override
	public void channelReadComplete(ChannelHandlerContext ctx) throws Exception {
		ctx.flush();
	}

	private void handlerWebSocketFrame(ChannelHandlerContext ctx, WebSocketFrame frame) {
		// 判断是否关闭链路的指令
		if (frame instanceof CloseWebSocketFrame) {
			handshaker.close(ctx.channel(), (CloseWebSocketFrame) frame.retain());
			return;
		}
		// 判断是否ping消息
		if (frame instanceof PingWebSocketFrame) {
			ctx.channel().write(
					new PongWebSocketFrame(frame.content().retain()));
			return;
		}
		// 本例程仅支持文本消息，不支持二进制消息
		if (!(frame instanceof TextWebSocketFrame)) {
			System.out.println("本例程仅支持文本消息，不支持二进制消息");
			throw new UnsupportedOperationException(String.format(
					"%s frame types not supported", frame.getClass().getName()));
		}
		// 返回应答消息
		String request = ((TextWebSocketFrame) frame).text();
		System.out.println("服务端收到：" + request);
		TextWebSocketFrame tws = new TextWebSocketFrame(new Date().toString()
				+ ctx.channel().id() + "：" + request);
		// 群发
		ChannelSupervise.send2All(tws);
		// 返回【谁发的发给谁】
		// ctx.channel().writeAndFlush(tws);
	}

	/**
	 * 唯一的一次http请求，用于创建websocket
	 */
	private void handleHttpRequest(ChannelHandlerContext ctx,
	                               FullHttpRequest req) {
		//要求Upgrade为websocket，过滤掉get/Post
		if (!req.decoderResult().isSuccess()
				|| (!"websocket".equals(req.headers().get("Upgrade")))) {
			//若不是websocket方式，则创建BAD_REQUEST的req，返回给客户端
			sendHttpResponse(ctx, req, new DefaultFullHttpResponse(
					HttpVersion.HTTP_1_1, HttpResponseStatus.BAD_REQUEST));
			return;
		}
		WebSocketServerHandshakerFactory wsFactory = new WebSocketServerHandshakerFactory(
				"ws://192.168.240.129:8082/websocket", null, false);
		handshaker = wsFactory.newHandshaker(req);
		if (handshaker == null) {
			WebSocketServerHandshakerFactory.sendUnsupportedVersionResponse(ctx.channel());
		} else {
			handshaker.handshake(ctx.channel(), req);
		}
	}

	/**
	 * 拒绝不合法的请求，并返回错误信息
	 */
	private static void sendHttpResponse(ChannelHandlerContext ctx,
	                                     FullHttpRequest req, DefaultFullHttpResponse res) {
		// 返回应答给客户端
		if (res.status().code() != 200) {
			ByteBuf buf = Unpooled.copiedBuffer(res.status().toString(),
					CharsetUtil.UTF_8);
			res.content().writeBytes(buf);
			buf.release();
		}
		ChannelFuture f = ctx.channel().writeAndFlush(res);
		// 如果是非Keep-Alive，关闭连接
		if (!isKeepAlive(req) || res.status().code() != 200) {
			f.addListener(ChannelFutureListener.CLOSE);
		}
	}
}
```

### 详解

```
继承SimpleChannelInboundHandler实现父类的方法 如请求进入 请求退出 消息接收的等
channelRead0中判断消息类型 看是握手请求 还是webSocket请求 分别处理
握手请求的话,处理后升级为websocket请求handleHttpRequest();
如果是websocket的话 直接处理业务逻辑;

```

### 管道消息的新增删除 及其群发的工具类

```java
**
 * 管道消息新增清除 及其群发
 */
public class ChannelSupervise {
	private static ChannelGroup GlobalGroup = new DefaultChannelGroup(GlobalEventExecutor.INSTANCE);
	private static ConcurrentMap<String, ChannelId> ChannelMap = new ConcurrentHashMap();

	public static void addChannel(Channel channel) {
		GlobalGroup.add(channel);
		ChannelMap.put(channel.id().asShortText(), channel.id());
	}

	public static void removeChannel(Channel channel) {
		GlobalGroup.remove(channel);
		ChannelMap.remove(channel.id().asShortText());
	}

	public static Channel findChannel(String id) {
		return GlobalGroup.find(ChannelMap.get(id));
	}

	public static void send2All(TextWebSocketFrame tws) {
		GlobalGroup.writeAndFlush(tws);
	}
}
```



## netty服务器处理链

```java
/**
 * Netty服务器处理链
 */
@Component
@Qualifier("nioWebSocketChannelInitializer")
public class NioWebSocketChannelInitializer extends ChannelInitializer<SocketChannel> {

	@Autowired
	private NioHttpAndWebSocketHandler httpRequestHandler;


	@Override
	protected void initChannel(SocketChannel ch) {
		//设置log监听器，并且日志级别为debug，方便观察运行流程
		ch.pipeline().addLast("logging", new LoggingHandler("INFO"));
		//设置解码器
		ch.pipeline().addLast("http-codec", new HttpServerCodec());
		//聚合器，使用websocket会用到
		ch.pipeline().addLast("aggregator", new HttpObjectAggregator(65536));
		//用于大数据的分区传输
		ch.pipeline().addLast("http-chunked", new ChunkedWriteHandler());
		//用于处理websocket, /ws为访问websocket时的uri
		ch.pipeline().addLast(new WebSocketServerProtocolHandler("/ws"));
		//自定义的业务handler 这种转发Http请求变为WwbSocket请求
		ch.pipeline().addLast( httpRequestHandler);
	}
}
```

## 启动netty服务器

```java

/**
 * 启动nettyServer服务器
 */
@Component
public class NioWebSocketServer {
	private Integer port = 8082;

	private void init() {
		System.out.println("正在启动websocket服务器");
		NioEventLoopGroup boss = new NioEventLoopGroup();
		NioEventLoopGroup work = new NioEventLoopGroup();
		try {
			long begin = System.currentTimeMillis();
			ServerBootstrap bootstrap = new ServerBootstrap();
			bootstrap.group(boss, work);
			bootstrap.channel(NioServerSocketChannel.class);
			//自定义业务handler
			bootstrap.childHandler(SpringContextUtil.getBean("nioWebSocketChannelInitializer"));
			Channel channel = bootstrap.bind(port).sync().channel();
			System.out.println("webSocket服务器启动成功");
			long end = System.currentTimeMillis();
			System.out.println("Netty Websocket服务器启动完成，耗时 " + (end - begin) + " ms，已绑定端口 " + port + " 阻塞式等候客户端连接");
			channel.closeFuture().sync();
		} catch (InterruptedException e) {
			e.printStackTrace();
			System.out.println("运行出错" + e);

		} finally {
			boss.shutdownGracefully();
			work.shutdownGracefully();
			System.out.println("websocket服务器已关闭");
		}
	}

	/**
	 * 启动初始化NettyServer
	 */
	public static void startNettyServer() {
		new NioWebSocketServer().init();
	}
}
```

### 这里需要注意的是 spring好像无法注入进去

所以需要新增一个工具类进行获取bean

```java

@Component
public class SpringContextUtil implements ApplicationContextAware {
	/**
	 * 获取spring容器，以访问容器中定义的其他bean
	 */
	private static ApplicationContext context;

	/**
	 * 实现ApplicationContextAware接口的回调方法，设置上下文环境
	 *
	 * @param context
	 */
	@Override
	public void setApplicationContext(ApplicationContext context)
			throws BeansException {
		SpringContextUtil.context = context;
	}

	/**
	 * @return ApplicationContext
	 */
	public static ApplicationContext getApplicationContext() {
		return context;
	}

	/**
	 * 获取对象
	 * 这里重写了bean方法，起主要作用
	 *
	 * @param beanName
	 * @return Object 一个以所给名字注册的bean的实例
	 * @throws BeansException
	 */

	public static <T> T getBean(String beanName) throws BeansException {
		return (T) context.getBean(beanName);
	}

	public static void destroy() {
		context = null;
	}

	public static String getMessage(String key) {
		return context.getMessage(key, null, Locale.getDefault());
	}

```



# web服务启动netty

```
@SpringBootApplication
public class NettyWebsocketDemoApplication {


	public static void main(String[] args) {
		SpringApplication.run(NettyWebsocketDemoApplication.class, args);

		//启动服务
		NioWebSocketServer.startNettyServer();
	}

}
```

