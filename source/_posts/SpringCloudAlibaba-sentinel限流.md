---
title: SpringCloudAlibaba-sentinel限流
date: 2020-04-22 14:36:28
tags: [SpringCloudAlibaba,sentinel]
---

# sentinel限流

基础使用

## [demo地址](https://github.com/AsummerCat/sentinel-demo)

## AbstractRule

```
在这个抽象类下面有多个实现类
1.FlowSlot  
   这个 slot 主要根据预设的资源的统计信息，按照固定的次序，依次生效。如果一个资源对应两条或者多条流控规则，则会根据如下次序依次检验，直到全部通过或者有一个规则生效为止

2.DegradeSlot
  这个 slot 主要针对资源的平均响应时间（RT）以及异常比率，来决定资源是否在接下来的时间被自动熔断掉。
  
3.SystemSlot
   这个 slot 会根据对于当前系统的整体情况，对入口资源的调用进行动态调配。其原理是让入口的流量和当前系统的预计容量达到一个动态平衡。
注意系统规则只对入口流量起作用（调用类型为 EntryType.IN），对出口流量无效。可通过 SphU.entry(res, entryType) 指定调用类型，如果不指定，默认是EntryType.OUT。

4.AuthorityRule 
黑白名单策略 限制模式，AUTHORITY_WHITE 为白名单模式，AUTHORITY_BLACK 为黑名单模式，默认为白名单模式。
  
```

<!--more-->

## 1.  初始化策略(硬编码)

可以在控制台 手动修改 默认是内存存储 也可以持久化

使用注解 和SphO,SphU都是埋点 可以修改的

## 限流策略

```
/**
	 * 限流规则策略
	 */
	private static void initFlowQpsRule() {
		//定义规则 可配置多个规则
		List<FlowRule> rules = new ArrayList<>();
		//1.配置资源名称
		FlowRule rule1 = new FlowRule("HelloWorld");
		// set limit qps to 20
		//2.配置限流数量
		rule1.setCount(100);
		//3.限流策略     QPS OR THREAD 限流策略可以选择根据qps 或者线程数
		rule1.setGrade(RuleConstant.FLOW_GRADE_QPS);
		//4.流量控制的效果  直接拒绝、Warm Up、匀速排队
		rule1.setControlBehavior(RuleConstant.CONTROL_BEHAVIOR_DEFAULT);
		//5.流控模式
		//STRATEGY_DIRECT = 0;  //direct 直接模式
		//STRATEGY_RELATE = 1;  //relate 关联
		//STRATEGY_CHAIN = 2;   //chain  链路
		rule1.setStrategy(RuleConstant.STRATEGY_DIRECT);
		//6. 受来源限制的应用程序名称 默认 default
		rule1.setLimitApp("default");
		//7.添加到list
		rules.add(rule1);
		//8.加载规则
		FlowRuleManager.loadRules(rules);

		System.out.println("初始化限流策略成功");
	}

```

## 降级策略

```
/**
	 * 降级规则策略
	 */
	private static void initDegradeRule() {
		List<DegradeRule> rules = new ArrayList<>();
		DegradeRule rule = new DegradeRule();
		//1.定义资源名
		rule.setResource("HelloWorld");
		// set threshold RT, 10 ms
		//2.阀值
		rule.setCount(1);
		//3.熔断策略，支持秒级 RT/秒级异常比例/分钟级异常数	秒级平均 RT 默认:秒级平均RT
		rule.setGrade(RuleConstant.DEGRADE_GRADE_RT);
		//4. 降级的时间 ,单位为s
		rule.setTimeWindow(10);
		//5. 滑动窗口 RT 模式下 1 秒内连续多少个请求的平均 RT 超出阈值方可触发熔断  默认5
		rule.setRtSlowRequestAmount(5);
		//6.异常熔断的触发最小请求数，请求数小于该值时即使异常比率超出阈值也不会熔断 默认5
		rule.setMinRequestAmount(5);
		rules.add(rule);
		DegradeRuleManager.loadRules(rules);
		System.out.println("初始化降级策略成功");
	}
```

 ## 系统保护规则策略

```
/**
	 * 系统保护规则策略
	 * 注意系统规则只针对入口资源（EntryType=IN）生效
	 */
	private static void initSystemRule() {
		List<SystemRule> rules = new ArrayList<>();
		SystemRule rule = new SystemRule();
		// max load is 3
		//1.load1 触发值，用于触发自适应控制阶段
		rule.setHighestSystemLoad(3.0);
		// max cpu usage is 60%
		//2.当前系统的 CPU 使用率（0.0-1.0）
		rule.setHighestCpuUsage(0.6);
		//33所有入口流量的平均响应时间
		rule.setAvgRt(10);
		//4.所有入口资源的 QPS
		rule.setQps(20);
		//5.入口流量的最大并发数
		rule.setMaxThread(10);

		rules.add(rule);
		SystemRuleManager.loadRules(Collections.singletonList(rule));
	}

```

## 黑白名单控制策略

```
/**
	 * 黑白名单控制策略
	 */
	private static void initAuthorityRule() {
		AuthorityRule rule = new AuthorityRule();
		//1.资源名，即限流规则的作用对象
		rule.setResource("HelloWorld");
		//2.限制模式，AUTHORITY_WHITE 为白名单模式，AUTHORITY_BLACK 为黑名单模式，默认为白名单模式。
		rule.setStrategy(RuleConstant.AUTHORITY_WHITE);
		//3.对应的黑名单/白名单，不同 origin 用 , 分隔，如 appA,appB。
		rule.setLimitApp("appA,appB");
		AuthorityRuleManager.loadRules(Collections.singletonList(rule));
	}
```

## 根据调用方限流

```
ContextUtil.enter(resourceName, origin) 方法中的origin 参数标明了调用身份。这些信息会在ClusterBuilderSlot 中统计。

流量规则中的limitApp 字段用于根据调用来源进行流量控制。该字段的值有以下三种选择，分别对应不同的场景：

1. default ：
     表示不区分调用者，来自任何调用者的请求都将进行限流统计。如果这个资源名的调用总和超过了这条规则定义的阈值，则出发限流。
2. {some_origin_name} : 
     表示针对特定的调用者，只有来自这个调用者的请求才会进行流量控制。例如NodeA 配置了一条针对调用者caller1 的规则，那么当且仅当来自caller1 对 NodeA 的请求才会触发流量控制。
3. other ：
         表示针对除{some_origin_name} 以外的其余调用方的流量进行流量控制。例如：资源NodeA 配置了一条针对调用者caller1 的限流规则，同时又配置了一条调用者为other 的规则，那么任意来自非caller1 对NodeA 的调用，都不能超过other这条规则定义的阈值。
同一资源名可以配置多条规则，规则生效的顺序为:{some_origin_name} > other > default.

```

```
/*定义根据调用者的流控规则*/
public static void initFlowRuleForCaller(){
    List<FlowRule> rules = new ArrayList<>();
    FlowRule rule = new FlowRule();
    //定义资源名
    rule.setResource("echo");
    //定义阈值类型
    rule.setGrade(RuleConstant.FLOW_GRADE_QPS);
    //定义阈值
    rule.setCount(2);
    //定义限制调用者
    rule.setLimitApp("caller");
    rules.add(rule);

    FlowRule rule1 = new FlowRule();
    rule1.setResource("echo");
    rule1.setGrade(RuleConstant.FLOW_GRADE_QPS);
    rule1.setLimitApp("other");
    rule1.setCount(3);
    rules.add(rule1);
    FlowRuleManager.loadRules(rules);
}


public static void testFlowRuleForCaller(){
  initFlowRuleForCaller();
  for (int i = 0; i < 5; i++) {
    ContextUtil.enter("c1","caller");
    Entry entry = null;
    try {
      entry = SphU.entry("echo");
      System.out.println("访问成功");
    } catch (BlockException e) {
      System.out.println("网络异常，请刷新！");
    }finally {
      if (entry != null){
        entry.exit();
      }
    }
  }
}
// =========测试结果：=========
/*
访问成功
访问成功
网络异常，请刷新！
网络异常，请刷新！
网络异常，请刷新！*/

// ===========将caller换成caller1测试，结果如下============
/*
访问成功
访问成功
访问成功
网络异常，请刷新！
网络异常，请刷新！*/

```

## 根据调用链路限流

表示示只有从入口 `你所指定的资源入口` 的调用才会记录到 `echo` 的限流统计当中，而不关心经 `Entrance2` 到来的调用。

 在配置规则的时候设置

```
 //定义入口资源 
  rule.setRefResource("你所指定的资源入口");
    //定义流控模式
  rule.setStrategy(RuleConstant.STRATEGY_CHAIN);
```

使用

```
 ContextUtil.enter("Entrance1"); 标记当前使用的入口是 Entrance1
```

## 关联流量

```
举例来说，read_db 和 write_db 这两个资源分别代表数据库读写，我们可以给 read_db 设置限流规则来达到写优先的目的：设置 FlowRule.strategy 为 RuleConstant.RELATE 同时设置 FlowRule.ref_identity 为 write_db。这样当写库操作过于频繁时，读数据的请求会被限流。
```



### 1.1 流量控制的效果

流量控制的效果包括以下几种：**直接拒绝**、**Warm Up**、**匀速排队**。对应 `FlowRule` 中的 `controlBehavior` 字段。

#### 1.1.1 直接拒绝（RuleConstant.CONTROL_BEHAVIOR_DEFAULT）

```
直接拒绝（RuleConstant.CONTROL_BEHAVIOR_DEFAULT）方式是默认的流量控制方式，当QPS超过任意规则的阈值后，新的请求就会被立即拒绝，拒绝方式为抛出FlowException。这种方式适用于对系统处理能力确切已知的情况下，比如通过压测确定了系统的准确水位时。
```

#### 1.1.2  Warm Up （RuleConstant.CONTROL_BEHAVIOR_WARM_UP）

```
Warm Up（RuleConstant.CONTROL_BEHAVIOR_WARM_UP）方式，即预热/冷启动方式。当系统长期处于低水位的情况下，当流量突然增加时，直接把系统拉升到高水位可能瞬间把系统压垮。通过"冷启动"，让通过的流量缓慢增加，在一定时间内逐渐增加到阈值上限，给冷系统一个预热的时间，避免冷系统被压垮。
```

#### 1.1.3  匀速排队RuleConstant.CONTROL_BEHAVIOR_RATE_LIMITER

```
匀速排队（RuleConstant.CONTROL_BEHAVIOR_RATE_LIMITER）方式会严格控制请求通过的间隔时间，也即是让请求以均匀的速度通过，对应的是漏桶算法
```

若不配置 `blockHandler`、`fallback` 等函数，则被流控降级时方法会直接抛出对应的 BlockException；若方法未定义 `throws BlockException` 则会被 JVM 包装一层 `UndeclaredThrowableException`。

## 2.配置限流方式

```
分为3种 
1. 手动设置  异常限流处理会抛出异常       -> SphU.entry
2. 手动设置  if限流处理 返回true和false  -> SphO.entry("HelloWorld");
3. 注解  注解限流 @SentinelResource  

```

### 2.1. 异常限流测试代码

```
/**
	 * 异常限流处理  entry=SphU.entry("HelloWorld");
	 * entry.exit();
	 * 必须成对出现
	 *
	 * @return
	 */
	@RequestMapping("index")
	public String index() {
		Entry entry = null;
		try {
			entry = SphU.entry("HelloWorld");
			return "HelloController.index 正常访问";
		} catch (BlockException e) {
			e.printStackTrace();
			return "HelloController.index 限流中";
		} finally {
			if (entry != null) {
				entry.exit();
			}
		}
	}
```

### 2.2 if限流处理

```

	/**
	 * if限流处理  SphO.entry("HelloWorld");
	 *
	 * @return
	 */
	@RequestMapping("index1")
	public String index1() {
		if (SphO.entry("HelloWorld")) {

			return "HelloController.index1 正常访问";
		} else {
			return "HelloController.index1 限流中";
		}
	}
```

### 2.3注解限流

方式一: 直接在当前类下写限流后 和降级后的方法

```
	/**
	 * 注解限流 @SentinelResource
     * 直接写在一个类中 对应限流 降级方法
	 *
	 * @return
	 */
	@RequestMapping("index3")
	@SentinelResource(value = "HelloWorld",
			blockHandler="block",
			fallback="fallback"
	)
	public String index3() {
		return "HelloController.index3 正常访问";
	}
	/**
	 * 处理限流或者降级
	 */
	public String block(BlockException e) {
		return "限流，或者降级了 block";
	}

	public String fallback(BlockException e) {
		return "限流，或者降级了 fallback";
	}
```

方式二 : 利用mock类去实现相同的内容 来降级

```
/**
	 * 注解限流 @SentinelResource
	 * blockHandler:blockHandlerClass中对应的异常处理方法名。参数类型和返回值必须和原方法一致
	 * blockHandlerClass：自定义限流逻辑处理类
	 * fallback: 对应降级方法
	 * fallbackClass: 相应降级策略
	 * exceptionsToIgnore：用于指定哪些异常被排除掉，不会计入异常统计中，也不会进入 fallback 逻辑中，而是会原样抛出。
	 *
	 * @return
	 */
	@RequestMapping("index2")
	@SentinelResource(value = "HelloWorld",
			blockHandler="index2",
			blockHandlerClass = HelloControllerMock.class,
			fallback="index2" ,
			fallbackClass = HelloControllerMock.class
	)
	public String index2() {
		return "HelloController.index2 正常访问";
	}
```

 mock类

```
/**
 * 限流业务处理
 */
public class HelloControllerMock {
	//注意对应的函数必需为 static 函数，否则无法解析
	public static String index2(BlockException e) {
		return "HelloController.index2 限流中";
	}
}

```

## 3.配置文件yml

```
server:
  port: 9090
spring:
  application:
    name: sentinel-demo
  cloud:
    sentinel:
      # 日志地址 默认地址为 C盘user下的log/scp
      log:
        dir: ./src/main/resources/logs/${appName}
      # 控制台地址
      transport:
        dashboard: localhost:9080
      # 开启控制台懒加载
      eager: false
```

其他更多配置

```
https://github.com/alibaba/Sentinel/wiki/%E5%90%AF%E5%8A%A8%E9%85%8D%E7%BD%AE%E9%A1%B9
```

## 4.控制台下载

**注意**: 集群资源汇总仅支持 500 台以下的应用集群，有大概 1 - 2 秒的延时。

### [官网地址](https://github.com/alibaba/Sentinel/releases)

默认情况下 直接打开jar ->路径指向localhost:8080 默认账号密码 sentinel :sentinel

sentinel-dashboard不像Nacos的服务端那样提供了外置的配置文件，比较容易修改参数。不过不要紧，由于sentinel-dashboard是一个标准的spring boot应用，所以如果要自定义端口号等内容的话，可以通过在启动命令中增加参数来调整，比如：`-Dserver.port=8888`。

```
java -jar sentinel-dashboard-1.7.2.jar
```

```
-Dsentinel.dashboard.auth.username=sentinel: 用于指定控制台的登录用户名为 sentinel；
-Dsentinel.dashboard.auth.password=123456: 用于指定控制台的登录密码为 123456；如果省略这两个参数，默认用户和密码均为 sentinel
-Dserver.servlet.session.timeout=7200: 用于指定 Spring Boot 服务端 session 的过期时间，如 7200 表示 7200 秒；60m 表示 60 分钟，默认为 30 分钟；
```

### 整合微服务

客户端需要引入 Transport 模块来与 Sentinel 控制台进行通信。您可以通过 `pom.xml` 引入 JAR 包:

如果是微服务的starter包的话 自动有了对应的通信包

```
<dependency>
    <groupId>com.alibaba.csp</groupId>
    <artifactId>sentinel-transport-simple-http</artifactId>
    <version>x.y.z</version>
</dependency>
```

```
yml配置文件中加入
spring.cloud.sentinel.transport.dashboard :localhost:8080
```

可以手动在控制台设置某一个资源的限流策略 实时生效 

![控制台](/img/2020-04-22/sentinel1.png)

## 5.跟Feign 整合

配置文件打开 Sentinel 对 Feign 的支持：feign.sentinel.enabled=true

```
加入 spring-cloud-starter-openfeign 依赖使 Sentinel starter 中的自动化配置类生效：

<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

### 测试

```
@FeignClient(name = "service-provider", fallback = EchoServiceFallback.class, configuration = FeignConfiguration.class)
public interface EchoService {
    @RequestMapping(value = "/echo/{str}", method = RequestMethod.GET)
    String echo(@PathVariable("str") String str);
}

class FeignConfiguration {
    @Bean
    public EchoServiceFallback echoServiceFallback() {
        return new EchoServiceFallback();
    }
}

class EchoServiceFallback implements EchoService {
    @Override
    public String echo(@PathVariable("str") String str) {
        return "echo fallback";
    }
}
```

## 6.使用Nacos存储限流规则(持久化)

Sentinel自身就支持了多种不同的数据源来持久化规则配置，目前包括以下几种方式：

- 文件配置
- [Nacos配置](http://blog.didispace.com/spring-cloud-alibaba-sentinel-2-1/)
- ZooKeeper配置
- [Apollo配置](http://blog.didispace.com/spring-cloud-alibaba-sentinel-2-2/)

添加整合包

```
<dependency>
    <groupId>com.alibaba.csp</groupId>
    <artifactId>sentinel-datasource-nacos</artifactId>
    <version>1.7.2</version>
    <scope>test</scope>
</dependency>
```

# 后面整合nacos的部分 新开一篇文章

