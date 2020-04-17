---
title: 基于guava实现的限流
date: 2020-03-19 08:34:33
tags:  [guava,java]
---

# 基于guava实现的限流

## demo

[demo地址](https://github.com/AsummerCat/guava_limit_demo)

## 导入pom

```java
     <!--guava工具包-->
        <dependency>
            <groupId>com.google.guava</groupId>
            <artifactId>guava</artifactId>
            <version>28.2-jre</version>
        </dependency>

        <!-- 切面声明-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-aop</artifactId>
        </dependency>

        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.12</version>
            <scope>provided</scope>
        </dependency>
```

## 主要部分

1.基于guava创建缓存+限流+惰性删除

2.再添加一个延迟队列 监控 实现 限流过期删除

```
1.缓存池
private static final Cache<Object, CurrentLimit> cache = CacheBuilder.newBuilder().maximumSize(100000).expireAfterAccess(15L, TimeUnit.DAYS).build();
2.限流
RateLimiter.create(limitInfo.getValue());

```

<!--more-->

# 以下部分使用aop完成

## 创建限流注解

```java
package com.linjingc.guava_limit_demo.requestLimitConfig.annotation;


import com.linjingc.guava_limit_demo.requestLimitConfig.Strategy.ReleaseTimeoutStrategy;
import com.linjingc.guava_limit_demo.requestLimitConfig.basicLimitType.LimitType;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

/**
 * 限流注解
 *
 * @author 一只写Bug的猫
 * @date 2020年3月16日17:32:01
 */
@Target(value = {ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
public @interface RequestLimit {

	/**
	 * 限流的速率
	 *
	 * @return
	 */
	double value() default 60d;

	/**
	 * 限流的名称 如果设置同样的name 表示多个方法共用一个限流池
	 */
	String name() default "";

	/**
	 * 限流类型
	 * 具体类型参考
	 * @see LimitType
	 *
	 * @return
	 */
	LimitType limitType() default LimitType.TokenBucketLimiter;

	/**
	 * 自定义业务key
	 *
	 * @return keys
	 */
	String[] keys() default {};
	/**
	 * 获取令牌失败的处理策略
	 *
	 * @see ReleaseTimeoutStrategy
	 */
	ReleaseTimeoutStrategy ReleaseTimeoutStrategy() default ReleaseTimeoutStrategy.NO_OPERATION;
    /**
	 * 过期时间
	 */
     long delayTime() default 0;
	/**
	 * 过期时间的类型 默认分钟
	 */
	 TimeUnit unit() default TimeUnit.MINUTES;
}

```

## 创建自定义异常

```java
/**
 * 自定义获取令牌失败 错误
 *
 * @author 一只写Bug的猫
 * @date 2019年8月8日18:16:08
 */
public class AcquireTimeoutException extends RuntimeException {

	public AcquireTimeoutException() {
	}

	public AcquireTimeoutException(String message) {
		super(message);
	}

	public AcquireTimeoutException(String message, Throwable cause) {
		super(message, cause);
	}
}

```

## 创建限流info的实体

```java


/**
 * 当前限流属性
 *
 * @author 一只写Bug的猫
 * @since 2019年8月8日18:19:18
 */
@Data
public class LimitInfo {

	public LimitInfo(String name, LimitType type, Double value, Long delayTime) {
		this.name = name;
		this.type = type;
		this.value = value;
		this.delayTime = delayTime;
	}

	public LimitInfo(String name, LimitType type, Double value) {
		this.name = name;
		this.type = type;
		this.value = value;
	}

	/**
	 * 限流的名称
	 */
	private String name;
	/**
	 * 限流的类型
	 */
	private LimitType type;

	/**
	 * 限流的速率
	 *
	 * @return
	 */
	private Double value;
	/**
	 * 延迟时间 -> 多久过期
	 * 毫秒
	 */
	private Long delayTime = 0L;

}

```

## 获取限流的令牌失败处理策略接口

```java
/**
 * 获取限流的令牌失败处理策略接口
 *
 * @author 一只写Bug的猫
 * @since 2019年8月8日18:19:18
 **/
public interface AcquireTokenFailureHandler {

	/**
	 * 处理
	 */
	void handle(JoinPoint joinPoint);
}

```

## 限流获取失败的 实现类

```java

/**
 * 限流获取失败的 策略接口
 *
 * @author 一只写Bug的猫
 * @date 2019年8月8日18:21:28
 **/
public enum ReleaseTimeoutStrategy implements AcquireTokenFailureHandler {

	/**
	 * 继续执行业务逻辑，不做任何处理
	 */
	NO_OPERATION() {
		@Override
		public void handle(JoinPoint joinPoint) {
			// do nothing
		}
	},
	/**
	 * 快速失败
	 */
	FAIL_FAST() {
		@Override
		public void handle(JoinPoint joinPoint) {
			MethodSignature signature = (MethodSignature) joinPoint.getSignature();
			String declaringTypeName = signature.getDeclaringTypeName();
			String controllerName;
			int lastIndex = declaringTypeName.lastIndexOf(".");
			if (lastIndex == -1) {
				controllerName = "";
			} else {
				controllerName = declaringTypeName.substring(lastIndex + 1);
			}
			String errorMsg = String.format("获取令牌失败 %s->%s方法,限流时间:%s,%s ", controllerName, signature.getMethod().getName(), LocalDateTime.now().toLocalDate(), "限流中.....");
			throw new AcquireTimeoutException(errorMsg);
		}
	}

}


```

## 创建获取限流名称前缀的策略接口

```
/**
 * 获取限流名称前缀的策略接口
 * @author 一只写Bug的猫
 * @date 2020年3月19日08:47:57
 **/
public interface LimitNameHandler {

	/**
	 * 获取限流名称前缀
	 *
	 * @param joinPoint 切面内容
	 */
	String prefixName(JoinPoint joinPoint);
}

```



## 创建令牌桶限流的枚举 实现策略接口

```java
/**
 * 令牌桶限流类型
 * 并且获取限流前缀名称
 */
public enum LimitType implements LimitNameHandler {
	/**
	 * 令牌桶限流 默认
	 */
	TokenBucketLimiter() {
		@Override
		public String prefixName(JoinPoint joinPoint) {
			String name = this.getClass().getName();
			return name;
		}
	},


	/**
	 * 根据ip限流
	 */
	IpLimiter() {
		@Override
		public String prefixName(JoinPoint joinPoint) {
			//获取访问的ip
			RequestAttributes requestAttributes = RequestContextHolder.getRequestAttributes();
			ServletRequestAttributes sra = (ServletRequestAttributes) requestAttributes;
			HttpServletRequest request = sra.getRequest();
			String ipAddress = getIpAddress(request);
			String name = this.getClass().getName() + ":" + ipAddress;
			return name;

		}
	},


	/**
	 * 根据IP和method限流
	 */
	IpAndMethodLimiter() {
		@Override
		public String prefixName(JoinPoint joinPoint) {
			String name = this.getClass().getName();
			return name;
		}
	};

	/**
	 * 获取用户真实IP地址，不使用request.getRemoteAddr();的原因是有可能用户使用了代理软件方式避免真实IP地址,
	 * <p>
	 * 可是，如果通过了多级反向代理的话，X-Forwarded-For的值并不止一个，而是一串IP值，究竟哪个才是真正的用户端的真实IP呢？
	 * 答案是取X-Forwarded-For中第一个非unknown的有效IP字符串。
	 * <p>
	 * 如：X-Forwarded-For：192.168.1.110, 192.168.1.120, 192.168.1.130,
	 * 192.168.1.100
	 * <p>
	 * 用户真实IP为： 192.168.1.110
	 */
	public String getIpAddress(HttpServletRequest request) {
		String ip = request.getHeader("x-forwarded-for");
		if (ip == null || ip.length() == 0 || "unknown".equalsIgnoreCase(ip)) {
			ip = request.getHeader("Proxy-Client-IP");
		}
		if (ip == null || ip.length() == 0 || "unknown".equalsIgnoreCase(ip)) {
			ip = request.getHeader("WL-Proxy-Client-IP");
		}
		if (ip == null || ip.length() == 0 || "unknown".equalsIgnoreCase(ip)) {
			ip = request.getHeader("HTTP_CLIENT_IP");
		}
		if (ip == null || ip.length() == 0 || "unknown".equalsIgnoreCase(ip)) {
			ip = request.getHeader("HTTP_X_FORWARDED_FOR");
		}
		if (ip == null || ip.length() == 0 || "unknown".equalsIgnoreCase(ip)) {
			ip = request.getRemoteAddr();
		}
		return ip;
	}

}

```

## 创建限流的工厂类

这里就是功能是

1.开启一个监控线程 监控超时

2.创建限流

```java

/**
 * 创建限流器的工厂
 *
 * @author 一只写Bug的猫
 * @date 2020年3月19日08:55:15
 */
@Component
public class CurrentLimitFactory implements InitializingBean {

	/**
	 * 缓存池
	 * 用来保存方法的限流策略
	 * 这边使用guava实现资源回收 避免过量增长
	 * 15天未读取就删除
	 */
	private static volatile Cache<Object, CurrentLimit> cache = CacheBuilder.newBuilder().maximumSize(100000).expireAfterAccess(15L, TimeUnit.DAYS).build();
	/**
	 * 手动过期删除策略
	 */
	private static ExecutorService executorService = Executors.newSingleThreadExecutor();
	private static volatile DelayQueue<DelayedTask> delayQueue = new DelayQueue<>();


	public CurrentLimit getLimit(LimitInfo limitInfo) {
		//判断缓存池是否有数据 没有数据就创建limit
		CurrentLimit limit = cache.getIfPresent(limitInfo.getName());
		if (Objects.isNull(limit)) {
			//创建limit
			return createLimit(limitInfo);
		}
		return limit;

	}

	/**
	 * 创建限流策略
	 *
	 * @param limitInfo
	 * @return
	 */
	private synchronized CurrentLimit createLimit(LimitInfo limitInfo) {
		CurrentLimit limit = cache.getIfPresent(limitInfo.getName());
		CurrentLimit currentLimit;
		//双重检测
		if (Objects.isNull(limit)) {
			currentLimit = new TokenBucketLimiter(limitInfo);
			cache.put(limitInfo.getName(), currentLimit);
				//添加超时任务
				addTimeoutTask(limitInfo);
		} else {
			currentLimit = limit;
		}
		return currentLimit;
	}


	/**
	 * 过期手动删除 这里先用类似redis的定时删除 和惰性删除模式
	 */
	private void addTimeoutTask(LimitInfo limitInfo){
		if(limitInfo.getDelayTime()>0){
		DelayedTask element = new DelayedTask(limitInfo.getDelayTime(),limitInfo);
		delayQueue.offer(element);
			System.out.println("添加超时任务到队列:->"+element.msg.getName());
		}
	}
	@Override
	public void afterPropertiesSet() throws Exception {
		/**
		 * 开启监控线程 监控超时移除
		 */
		executorService.execute(() -> {
			System.out.println("启动监控限流超时的队列线程启动中。。。。。。。。。。");
					while (true) {
						DelayedTask element;
						try {
							element = delayQueue.take();
							//移除过期key
							System.out.println("开始移除key:->"+element.msg.getName());
							cache.invalidate(element.msg.getName());
						} catch (InterruptedException e) {
							e.printStackTrace();
						}
					}
				}
		);
	}


	/**
	 * 延时队列的内容对象 过期删除
	 */
	static class DelayedTask implements Delayed {
		private final long delay; //延迟时间
		private final long expire;  //到期时间
		private final LimitInfo msg;   //数据
		private final long now; //创建时间

		public DelayedTask(long delay, LimitInfo msg) {
			this.delay = delay;
			this.msg = msg;
			expire = System.currentTimeMillis() + delay;    //到期时间 = 当前时间+延迟时间
			now = System.currentTimeMillis();
		}

		/**
		 * 需要实现的接口，获得延迟时间   用过期时间-当前时间
		 *
		 * @param unit
		 * @return
		 */
		@Override
		public long getDelay(TimeUnit unit) {
			//根据过期时间-当前时间
			return unit.convert(this.expire - System.currentTimeMillis(), TimeUnit.MILLISECONDS);
		}

		/**
		 * 用于延迟队列内部比较排序   当前时间的延迟时间 - 比较对象的延迟时间
		 *
		 * @param o
		 * @return
		 */
		@Override
		public int compareTo(Delayed o) {
			return (int) (this.getDelay(TimeUnit.MILLISECONDS) - o.getDelay(TimeUnit.MILLISECONDS));
		}

		@Override
		public String toString() {
			final StringBuilder sb = new StringBuilder("DelayedTask{");
			sb.append("delay=").append(delay);
			sb.append(", expire=").append(expire);
			sb.append(", msg='").append(msg.getName()).append('\'');
			sb.append(", now=").append(now);
			sb.append('}');
			return sb.toString();
		}
	}
}
```

## 限流接口

```java
/**
 * 限流实现接口
 *
 * @author 一只写Bug的猫
 */
public interface CurrentLimit {

	/**
	 * 限流
	 *
	 * @return
	 */
	boolean acquire();
}
```

## 限流接口实现类

```java
/**
 * 使用限流
 * 基于令牌桶实现
 */
public class TokenBucketLimiter implements CurrentLimit {
	private RateLimiter limiter;
	private LimitInfo limitInfo;

	public TokenBucketLimiter(LimitInfo limitInfo) {
		this.limitInfo = limitInfo;
		this.limiter = RateLimiter.create(limitInfo.getValue());
	}

	@Override
	public boolean acquire() {
		return limiter.tryAcquire();
	}
}

```



## 创建生成限流器的提供者 生成limitInfo类

```java
/**
 * 获取用户定义业务key
 * 生成限流info的内容
 *
 * @author 一只写Bug的猫
 * @date 2020年3月19日08:45:32
 */
@Component
public class BusinessKeyProvider {

	public LimitInfo get(JoinPoint joinPoint, RequestLimit requestLimit) {
		//获取到切面的信息
		MethodSignature signature = (MethodSignature) joinPoint.getSignature();
		//获取到限流类型 ,并且获取到前缀名称
		LimitType type = requestLimit.limitType();
		//根据自定义业务key 获取keyName
		String businessKeyName = getKeyName(joinPoint, requestLimit);
		//根据自定义name配置 如果存在name 则使用name,否则使用方法名当做name
		String limitName = type.prefixName(joinPoint) + ":" + getName(requestLimit.name(), signature) + businessKeyName;
		//根据过期时间添加 时间戳
		if(requestLimit.delayTime()>0){
			//计算毫秒
			long delayMillis = requestLimit.unit().toMillis(requestLimit.delayTime());
			return new LimitInfo(limitName, type, requestLimit.value(),delayMillis);
		}else{
			return new LimitInfo(limitName, type, requestLimit.value());
		}
	}


	private ParameterNameDiscoverer nameDiscoverer = new DefaultParameterNameDiscoverer();

	private ExpressionParser parser = new SpelExpressionParser();


	/**
	 * 获取限流的名称
	 *
	 * @param annotationName
	 * @param signature
	 * @return
	 */
	private String getName(String annotationName, MethodSignature signature) {
		//如果keyname没有设置 则返回方法名称
		if (annotationName.isEmpty()) {
			return String.format("%s.%s", signature.getDeclaringTypeName(), signature.getMethod().getName());
		} else {
			return annotationName;
		}
	}


	public String getKeyName(JoinPoint joinPoint, RequestLimit requestLimit) {
		List<String> keyList = new ArrayList<>();
		Method method = getMethod(joinPoint);
		//获取方法RequestLimit注解上的自定义keys
		List<String> definitionKeys = getSpelDefinitionKey(requestLimit.keys(), method, joinPoint.getArgs());
		keyList.addAll(definitionKeys);
		//进行拼接
		return StringUtils.collectionToDelimitedString(keyList, "", "-", "");
	}

	/**
	 * 获取到切到的当前方法
	 */
	private Method getMethod(JoinPoint joinPoint) {
		MethodSignature signature = (MethodSignature) joinPoint.getSignature();
		Method method = signature.getMethod();
		if (method.getDeclaringClass().isInterface()) {
			try {
				method = joinPoint.getTarget().getClass().getDeclaredMethod(signature.getName(), method.getParameterTypes());
			} catch (Exception e) {
				e.printStackTrace();
			}
		}
		return method;
	}

	/**
	 * 获取方法RequestLimit注解上的自定义keys
	 */
	private List<String> getSpelDefinitionKey(String[] definitionKeys, Method method, Object[] parameterValues) {
		List<String> definitionKeyList = new ArrayList<>();
		for (String definitionKey : definitionKeys) {
			if (definitionKey != null && !definitionKey.isEmpty()) {
				EvaluationContext context = new MethodBasedEvaluationContext(null, method, parameterValues, nameDiscoverer);
				String key = parser.parseExpression(definitionKey).getValue(context).toString();
				definitionKeyList.add(key);
			}
		}
		return definitionKeyList;
	}

}

```

## aop切面 及其异常处理

```java
/**
 * 限流切面类
 *
 * @author 一只写Bug的猫
 * @date 2020年3月19日08:46:24
 */
@Aspect
@Component  //声明首先加载入spring
@Order(0)
public class CurrentLimitAop {
	@Autowired
	BusinessKeyProvider businessKeyProvider;
	@Autowired
	CurrentLimitFactory currentLimitFactory;

	@Around(value = "@annotation(requestLimit)")
	public Object around(ProceedingJoinPoint joinPoint, RequestLimit requestLimit) throws Throwable {

		//获取出限流的基础信息
		LimitInfo limitInfo = businessKeyProvider.get(joinPoint, requestLimit);
		//根据工厂模式 获取到CurrentLimit
		CurrentLimit currentLimit = currentLimitFactory.getLimit(limitInfo);

		if (!currentLimit.acquire()) {
			//获取令牌失败的处理策略
			requestLimit.ReleaseTimeoutStrategy().handle(joinPoint);
		}
		return joinPoint.proceed();
	}


	/**
	 * 异常处理
	 *
	 * @param point
	 * @param requestLimit
	 * @param ex
	 */
	@AfterThrowing(value = "@annotation(requestLimit)", throwing = "ex")
	public void afterReturning(JoinPoint point, RequestLimit requestLimit, Exception ex) {
//		String methodName = point.getSignature().getName();
//		List<Object> args = Arrays.asList(point.getArgs());
//		System.out.println("连接点方法为：" + methodName + ",参数为：" + args + ",异常为：" + ex);

		//判断异常是否为限流导致的
		if (ex instanceof AcquireTimeoutException) {
			RequestAttributes requestAttributes = RequestContextHolder.getRequestAttributes();
			ServletRequestAttributes sra = (ServletRequestAttributes) requestAttributes;
			HttpServletResponse response = sra.getResponse();
			response.setCharacterEncoding("UTF-8");
			response.setContentType("application/json; charset=utf-8");
			try (ServletOutputStream out = response.getOutputStream()) {
				out.write(ex.getMessage().getBytes(StandardCharsets.UTF_8));
				out.flush();
			} catch (IOException e) {
				e.printStackTrace();
			}
		} else {
			// do nothing
		}

	}
}

```

