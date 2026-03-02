---
title: springboot整合redis实现分布式锁三redisson注解版本
date: 2019-08-13 23:21:10
tags: [redis,SpringBoot,分布式锁,redisson]
---

# springboot整合redis实现分布式锁三redisson注解版本

[demo地址](https://github.com/AsummerCat/redis-lock)

# 老规矩 导入pom

```
<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-configuration-processor</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-autoconfigure</artifactId>
        </dependency>

        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
         <!-- 切面声明-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-aop</artifactId>
        </dependency>
        <dependency>
            <groupId>org.redisson</groupId>
            <artifactId>redisson</artifactId>
            <version>3.11.2</version>
        </dependency>
```

<!--more-->

# 创建redisson配置类

```
/**
 * redis 锁 redisson配置
 * @author cxc
 * @date 2019年8月9日18:00:08
 */
@Configuration
public class RedissonConfig {

    @Bean
    public RedissonClient redisson() throws IOException {
        // 本例子使用的是yaml格式的配置文件，读取使用Config.fromYAML，如果是Json文件，则使用Config.fromJSON
        Config config = Config.fromYAML(RedissonConfig.class.getClassLoader().getResource("redisson-config.yml"));
        return Redisson.create(config);
    }
}
```

## 新建 redisson-config.yml

```
#Redisson配置 https://github.com/redisson/redisson/wiki/2.-%E9%85%8D%E7%BD%AE%E6%96%B9%E6%B3%95
singleServerConfig:
  address: "redis://112.74.43.136:6379"
  password: jingbaobao
  clientName: null
  database: 7 #选择使用哪个数据库0~15
  idleConnectionTimeout: 10000
  pingTimeout: 1000
  connectTimeout: 10000
  timeout: 3000
  retryAttempts: 3
  retryInterval: 1500
  reconnectionTimeout: 3000
  failedAttempts: 3
  subscriptionsPerConnection: 5
  subscriptionConnectionMinimumIdleSize: 1
  subscriptionConnectionPoolSize: 50
  connectionMinimumIdleSize: 32
  connectionPoolSize: 64
  dnsMonitoringInterval: 5000
  #dnsMonitoring: false

threads: 0
nettyThreads: 0
codec:
  class: "org.redisson.codec.JsonJacksonCodec"
transportMode: "NIO"
```

# 现在就开始创建注解 aop的内容了

基本步骤: catLock注解 标注加锁方法 -> lockKey 参数注解名称 

 aop 切面 加锁 释放锁    after处理完毕 释放锁 -> 异常 释放锁

异常机制 自定义异常 加锁超时 释放超时

根据工厂类 实现具体的锁类型 



## 创建 CatLock注解

```
/**
 * 方法级别
 * 加锁注解
 *
 * @author cxc
 * @date 2019年8月8日17:56:15
 */
@Target(value = {ElementType.METHOD})
@Retention(value = RetentionPolicy.RUNTIME)
public @interface CatLock {
    /**
     * 锁的名称
     *
     * @return name
     */
    String name() default "";

    /**
     * 锁类型，默认可重入锁
     *
     * @return lockType
     */
    LockType lockType() default LockType.Reentrant;

    /**
     * 尝试加锁，最多等待时间
     *
     * @return waitTime
     */
    long waitTime() default Long.MIN_VALUE;

    /**
     * 上锁以后xxx秒自动解锁
     *
     * @return leaseTime
     */
    long leaseTime() default Long.MIN_VALUE;

    /**
     * 自定义业务key
     *
     * @return keys
     */
    String[] keys() default {};

    /**
     * 加锁超时的处理策略
     *
     * @return lockTimeoutStrategy
     */
    LockTimeoutStrategy lockTimeoutStrategy() default LockTimeoutStrategy.NO_OPERATION;

    /**
     * 自定义加锁超时的处理策略
     *
     * @return customLockTimeoutStrategy
     */
    String customLockTimeoutStrategy() default "";

    /**
     * 释放锁时已超时的处理策略
     *
     * @return releaseTimeoutStrategy
     */
    ReleaseTimeoutStrategy releaseTimeoutStrategy() default ReleaseTimeoutStrategy.NO_OPERATION;

    /**
     * 自定义释放锁时已超时的处理策略
     *
     * @return customReleaseTimeoutStrategy
     */
    String customReleaseTimeoutStrategy() default "";

}

```

## 创建参数key 

```
/**
 * 这个注解用来标记 参数 使用key lock
 *
 * @author cxc
 * @date 2019年8月8日17:52:34
 */
@Target(value = {ElementType.PARAMETER, ElementType.TYPE})
@Retention(value = RetentionPolicy.RUNTIME)
public @interface LockKey {
    String value() default "";
}

```

# 这边就构建一个 工厂 来创建各种的基本锁

## 创建一个锁工厂

```
/**
 * 工厂模式 根据lock的类型自动加载 对应的锁类型
 *
 * @author cxc
 * @date 2019年8月8日17:50:38
 */
@Log4j2
@Component
public class LockFactory {
    @Autowired
    private RedissonClient redissonClient;

    public Lock getLock(LockInfo lockInfo) {
        switch (lockInfo.getType()) {
            case Fair:
                return new FairLock(redissonClient, lockInfo);
            case Read:
                return new ReadLock(redissonClient, lockInfo);
            case Write:
                return new WriteLock(redissonClient, lockInfo);
            default:
                return new ReentrantLock(redissonClient, lockInfo);
        }
    }

}

```

## 这边是一个标记锁一定实现的两个方法 加锁解锁

```
package com.linjingc.annotationredissonlock.lock.basiclock;

/**
 * 锁接口
 * 自定义的锁类型都需要实现该接口的内容
 *
 * @author cxc
 * @date 2019年8月8日17:59:04
 */
public interface Lock {

    /**
     * 加锁
     *
     * @return
     */
    boolean acquire();

    /**
     * 解锁
     *
     * @return
     */
    boolean release();
}

```



## 公平锁

```
/**
 * 公平锁
 * @author cxc
 * @date 2019年8月8日17:57:57
 */
public class FairLock implements Lock {

    private RLock rLock;

    private final LockInfo lockInfo;

    private RedissonClient redissonClient;

    public FairLock(RedissonClient redissonClient, LockInfo info) {
        this.redissonClient = redissonClient;
        this.lockInfo = info;
    }

    @Override
    public boolean acquire() {
        try {
            rLock = redissonClient.getFairLock(lockInfo.getName());
            return rLock.tryLock(lockInfo.getWaitTime(), lockInfo.getLeaseTime(), TimeUnit.SECONDS);
        } catch (InterruptedException e) {
            return false;
        }
    }

    @Override
    public boolean release() {
        if (rLock.isHeldByCurrentThread()) {

            try {
                return rLock.forceUnlockAsync().get();
            } catch (InterruptedException e) {
                return false;
            } catch (ExecutionException e) {
                return false;
            }
        }
        return false;
    }
}

```

## 读锁

```
/**
 * 读锁
 *
 * @author cxc
 * @date 2019年8月8日17:59:38
 */
public class ReadLock implements Lock {

    private RReadWriteLock rLock;

    private final LockInfo lockInfo;

    private RedissonClient redissonClient;

    public ReadLock(RedissonClient redissonClient, LockInfo info) {
        this.redissonClient = redissonClient;
        this.lockInfo = info;
    }

    @Override
    public boolean acquire() {
        try {
            rLock = redissonClient.getReadWriteLock(lockInfo.getName());
            return rLock.readLock().tryLock(lockInfo.getWaitTime(), lockInfo.getLeaseTime(), TimeUnit.SECONDS);
        } catch (InterruptedException e) {
            return false;
        }
    }

    @Override
    public boolean release() {
        if (rLock.readLock().isHeldByCurrentThread()) {
            try {
                return rLock.readLock().forceUnlockAsync().get();
            } catch (InterruptedException e) {
                return false;
            } catch (ExecutionException e) {
                return false;
            }
        }

        return false;
    }
}


```

## 写锁

```
/**
 * 写锁
 *
 * @author cxc
 * @date 2019年8月8日18:00:07
 */
public class WriteLock implements Lock {

    private RReadWriteLock rLock;

    private final LockInfo lockInfo;

    private RedissonClient redissonClient;

    public WriteLock(RedissonClient redissonClient, LockInfo info) {
        this.redissonClient = redissonClient;
        this.lockInfo = info;
    }

    @Override
    public boolean acquire() {
        try {
            rLock = redissonClient.getReadWriteLock(lockInfo.getName());
            return rLock.writeLock().tryLock(lockInfo.getWaitTime(), lockInfo.getLeaseTime(), TimeUnit.SECONDS);
        } catch (InterruptedException e) {
            return false;
        }
    }

    @Override
    public boolean release() {
        if (rLock.writeLock().isHeldByCurrentThread()) {
            try {
                return rLock.writeLock().forceUnlockAsync().get();
            } catch (InterruptedException | ExecutionException e) {
                return false;
            }
        }
        return false;
    }
}


```

## 可重入锁

```
/**
 * 可重入锁
 *
 * @author cxc
 * @date 2019年8月8日17:59:54
 */
public class ReentrantLock implements Lock {

    private RLock rLock;

    private final LockInfo lockInfo;

    private RedissonClient redissonClient;

    public ReentrantLock(RedissonClient redissonClient, LockInfo lockInfo) {
        this.redissonClient = redissonClient;
        this.lockInfo = lockInfo;
    }

    @Override
    public boolean acquire() {
        try {
            rLock = redissonClient.getLock(lockInfo.getName());
            return rLock.tryLock(lockInfo.getWaitTime(), lockInfo.getLeaseTime(), TimeUnit.SECONDS);
        } catch (InterruptedException e) {
            return false;
        }
    }

    @Override
    public boolean release() {
        if (rLock.isHeldByCurrentThread()) {
            try {
                return rLock.forceUnlockAsync().get();
            } catch (InterruptedException e) {
                return false;
            } catch (ExecutionException e) {
                return false;
            }
        }
        return false;
    }

    public String getKey() {
        return this.lockInfo.getName();
    }
}


```

这边就完成了基本锁类型的构造了





# 创建一个redisLock配置文件 用来设置默认过期时间 等待时间之类的信息

后期这个可以动态加载 yml里面的信息来更新配置文件

```
package com.linjingc.annotationredissonlock.lock.config;

import lombok.Data;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.Configuration;

/**
 * Redis 锁 自定义配置类
 *
 * @author cxc
 * @date 2019年8月8日18:00:25
 */
@Data
@Configuration
@ConfigurationProperties(prefix = RedisLockConfig.PREFIX)
public class RedisLockConfig {

    public static final String PREFIX = "linjingc.lock";
    //redisson
    private String address;
    private String password;
    //    private int database = 15;
    private ClusterServer clusterServer;
    private String codec = "org.redisson.codec.JsonJacksonCodec";
    //lock
    private long waitTime = 60;
    private long leaseTime = 60;


    public static class ClusterServer {

        private String[] nodeAddresses;

        public String[] getNodeAddresses() {
            return nodeAddresses;
        }

        public void setNodeAddresses(String[] nodeAddresses) {
            this.nodeAddresses = nodeAddresses;
        }
    }
}


```

# 创建一个LOCKINFO类

 保存锁的基本信息

```
package com.linjingc.annotationredissonlock.lock.model;

import com.linjingc.annotationredissonlock.lock.LockType;
import lombok.Data;

/**
 * 锁基本 信息
 *
 * @author cxc
 * @date 2019年8月8日17:13:18
 */
@Data
public class LockInfo {

    /**
     * 锁类型
     */
    private LockType type;
    /**
     * 锁名称
     */
    private String name;
    /**
     * 等待时间
     */
    private long waitTime;
    /**
     * 续约时间 ->处理时间 达到该时间会自动解锁
     */
    private long leaseTime;

    public LockInfo() {
    }

    public LockInfo(LockType type, String name, long waitTime, long leaseTime) {
        this.type = type;
        this.name = name;
        this.waitTime = waitTime;
        this.leaseTime = leaseTime;
    }

}


```

```
package com.linjingc.annotationredissonlock.lock;

public enum LockType {
    /**
     * 可重入锁
     */
    Reentrant,
    /**
     * 公平锁
     */
    Fair,
    /**
     * 读锁
     */
    Read,
    /**
     * 写锁
     */
    Write,

    /**
     * 红锁
     */

    /**
     * 联锁
     */
    MultiLock,


    /**
     * 红锁
     */
    RedLock;

    LockType() {
    }

}

```



# 下一步 创建 锁提供者  + 业务key处理生成 + aop切面类

## 创建业务key生成处理类

```
/**
 * 获取用户定义业务key
 *
 * @author cxc
 * @date 2019年08月08日20:52:40
 */
@Component
public class BusinessKeyProvider {

    private ParameterNameDiscoverer nameDiscoverer = new DefaultParameterNameDiscoverer();

    private ExpressionParser parser = new SpelExpressionParser();

    public String getKeyName(ProceedingJoinPoint joinPoint, CatLock catLock) {
        List<String> keyList = new ArrayList<>();
        Method method = getMethod(joinPoint);
        //获取方法CatLock注解上的自定义keys
        List<String> definitionKeys = getSpelDefinitionKey(catLock.keys(), method, joinPoint.getArgs());
        keyList.addAll(definitionKeys);
        //获取参数注解LockKey 上的内容
        List<String> parameterKeys = getParameterKey(method.getParameters(), joinPoint.getArgs());
        keyList.addAll(parameterKeys);
        //进行拼接
        return StringUtils.collectionToDelimitedString(keyList, "", "-", "");
    }

    private Method getMethod(ProceedingJoinPoint joinPoint) {
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
     * 获取方法CatLock注解上的自定义keys
     *
     * @param definitionKeys
     * @param method
     * @param parameterValues
     * @return
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


    /**
     * 获取参数注解LockKey 上的内容
     *
     * @param parameters
     * @param parameterValues
     * @return
     */
    private List<String> getParameterKey(Parameter[] parameters, Object[] parameterValues) {
        List<String> parameterKey = new ArrayList<>();
        //遍历参数
        for (int i = 0; i < parameters.length; i++) {
            //参数上带有注解
            if (parameters[i].getAnnotation(LockKey.class) != null) {
                LockKey keyAnnotation = parameters[i].getAnnotation(LockKey.class);
                //注解的内容不为空
                if (keyAnnotation.value().isEmpty()) {
                    parameterKey.add(parameterValues[i].toString());
                } else {
                    StandardEvaluationContext context = new StandardEvaluationContext(parameterValues[i]);
                    String key = parser.parseExpression(keyAnnotation.value()).getValue(context).toString();
                    parameterKey.add(key);
                }
            }
        }
        return parameterKey;
    }
}


```

## 创建 锁提供者

```
/**
 * 锁提供者  创建锁的相关信息都在这里生成
 *
 * @author cxc
 * @date 2019年8月9日18:02:25
 */
@Component
@Log4j2
public class LockInfoProvider {

    /**
     * 锁的key前缀
     */
    public static final String LOCK_NAME_PREFIX = "lock";
    public static final String LOCK_NAME_SEPARATOR = ".";


    @Autowired
    private RedisLockConfig redisLockConfig;

    /**
     * 自定义业务key
     */
    @Autowired
    private BusinessKeyProvider businessKeyProvider;


    /***
     * 获取锁信息
     * 锁的名称 = 前缀+(方法名 或 锁名) +自定义key
     * @param joinPoint
     * @param catLock
     * @return
     */
    public LockInfo get(ProceedingJoinPoint joinPoint, CatLock catLock) {
        //获取到切面的信息
        MethodSignature signature = (MethodSignature) joinPoint.getSignature();
        //获取到锁类型
        LockType type = catLock.lockType();
        //根据自定义业务key 获取keyName
        String businessKeyName = businessKeyProvider.getKeyName(joinPoint, catLock);
        //拼接lockName
        String lockName = LOCK_NAME_PREFIX + LOCK_NAME_SEPARATOR + getName(catLock.name(), signature) + businessKeyName;
        //获取等待时间 不设置则根据keyConfig的生成
        long waitTime = getWaitTime(catLock);
        //获取持有时间 不设置则根据keyConfig的生成
        long leaseTime = getLeaseTime(catLock);
        //如果持有时间设置为-1 表示不会过期
        if (leaseTime == -1 && log.isWarnEnabled()) {
            log.warn("Trying to acquire Lock({}) with no expiration, " +
                    "Klock will keep prolong the lock expiration while the lock is still holding by current thread. " +
                    "This may cause dead lock in some circumstances.", lockName);
        }

        //实例化锁
        return new LockInfo(type, lockName, waitTime, leaseTime);
    }


    /**
     * 获取锁名称
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


    /**
     * 如果默认是最大等待时间 则使用配置项内的时间 否则 使用自定义的时间
     *
     * @param lock
     * @return
     */
    private long getWaitTime(CatLock lock) {
        return lock.waitTime() == Long.MIN_VALUE ?
                redisLockConfig.getWaitTime() : lock.waitTime();
    }

    /**
     * 如果默认是最大续约时间 则使用配置项内的时间 否则 使用自定义的时间
     *
     * @param lock
     * @return
     */
    private long getLeaseTime(CatLock lock) {
        return lock.leaseTime() == Long.MIN_VALUE ?
                redisLockConfig.getLeaseTime() : lock.leaseTime();
    }
}


```

## 创建注解切面类

```
/**
 * redis锁切面类
 * 用来包裹方法使用
 *
 * @author cxc
 * @date 2019年8月8日17:19:55
 */
@Aspect
@Component
//声明首先加载入spring
@Order(0)
@Log4j2
public class CatLockAspectAop {
    @Autowired
    LockFactory lockFactory;
    @Autowired
    private LockInfoProvider lockInfoProvider;


    /**
     * 获取当前线程获取到的锁
     */
    private ThreadLocal<Lock> currentThreadLock = new ThreadLocal<>();
    /**
     * 该锁是否已经处理释放操作
     */
    private ThreadLocal<LockRes> currentThreadLockRes = new ThreadLocal<>();

    /**
     * 方法 环绕  加锁
     *
     * @param joinPoint 切面
     * @param catLock   锁类型
     * @return
     * @throws Throwable
     */
    @Around(value = "@annotation(catLock)")
    public Object around(ProceedingJoinPoint joinPoint, CatLock catLock) throws Throwable {
        //获取切面信息
        //MethodSignature signature = (MethodSignature) joinPoint.getSignature();
        //获取自定义锁信息
        LockInfo lockInfo = lockInfoProvider.get(joinPoint, catLock);
        //设置当前锁状态
        currentThreadLockRes.set(new LockRes(lockInfo, false));
        //根据工厂模式 获取到Lock
        Lock lock = lockFactory.getLock(lockInfo);
        //加锁
        boolean carryLock = lock.acquire();
        //获取锁时的超时处理
        if (!carryLock) {
            if (log.isWarnEnabled()) {
                log.warn("Timeout while acquiring Lock({})", lockInfo.getName());
            }

            //如果有自定义的超时策略
            if (!StringUtils.isEmpty(catLock.customLockTimeoutStrategy())) {
                return handleCustomLockTimeout(catLock.customLockTimeoutStrategy(), joinPoint);
            } else {
                //否则使用配置的超时处理策略
                catLock.lockTimeoutStrategy().handle(lockInfo, lock, joinPoint);
            }
        }
        currentThreadLock.set(lock);
        return joinPoint.proceed();
    }


    /**
     * 方法执行完毕 释放锁
     *
     * @param joinPoint
     * @param catLock
     * @throws Throwable
     */
    @AfterReturning(value = "@annotation(catLock)")
    public void afterReturning(JoinPoint joinPoint, CatLock catLock) throws Throwable {
        //释放锁
        releaseLock(catLock, joinPoint);
        //清理线程副本
        cleanUpThreadLocal();
    }

    /**
     * 切面 异常处理
     *
     * @param joinPoint
     * @param catLock
     * @param ex
     * @throws Throwable
     */
    @AfterThrowing(value = "@annotation(catLock)", throwing = "ex")
    public void afterThrowing(JoinPoint joinPoint, CatLock catLock, Throwable ex) throws Throwable {
        //释放锁
        releaseLock(catLock, joinPoint);
        //清理线程副本
        cleanUpThreadLocal();
        throw ex;
    }


    /**
     * 释放锁 避免重复释放锁
     * 如: 执行完毕释放一次 throw时又释放一次
     */
    private void releaseLock(CatLock catLock, JoinPoint joinPoint) throws Throwable {
        LockRes lockRes = currentThreadLockRes.get();
        //未执行过释放锁操作
        if (!lockRes.getUseState()) {
            boolean releaseRes = currentThreadLock.get().release();
            // avoid release lock twice when exception happens below
            lockRes.setUseState(true);
            if (!releaseRes) {
                handleReleaseTimeout(catLock, lockRes.getLockInfo(), joinPoint);
            }
        }
    }

    /**
     * 处理自定义加锁超时
     */
    private Object handleCustomLockTimeout(String lockTimeoutHandler, JoinPoint joinPoint) throws Throwable {

        // prepare invocation context
        Method currentMethod = ((MethodSignature) joinPoint.getSignature()).getMethod();
        Object target = joinPoint.getTarget();
        Method handleMethod = null;
        try {
            handleMethod = joinPoint.getTarget().getClass().getDeclaredMethod(lockTimeoutHandler, currentMethod.getParameterTypes());
            handleMethod.setAccessible(true);
        } catch (NoSuchMethodException e) {
            throw new IllegalArgumentException("Illegal annotation param customLockTimeoutStrategy", e);
        }
        Object[] args = joinPoint.getArgs();

        // invoke
        Object res = null;
        try {
            res = handleMethod.invoke(target, args);
        } catch (IllegalAccessException e) {
            throw new CatLockInvocationException("Fail to invoke custom lock timeout handler: " + lockTimeoutHandler, e);
        } catch (InvocationTargetException e) {
            throw e.getTargetException();
        }

        return res;
    }


    /**
     * 处理释放锁时已超时
     */
    private void handleReleaseTimeout(CatLock catLock, LockInfo lockInfo, JoinPoint joinPoint) throws Throwable {

        if (log.isWarnEnabled()) {
            log.warn("Timeout while release Lock({})", lockInfo.getName());
        }

        if (!StringUtils.isEmpty(catLock.customReleaseTimeoutStrategy())) {

            handleCustomReleaseTimeout(catLock.customReleaseTimeoutStrategy(), joinPoint);

        } else {
            catLock.releaseTimeoutStrategy().handle(lockInfo);
        }
    }

    /**
     * 处理自定义释放锁时已超时
     */
    private void handleCustomReleaseTimeout(String releaseTimeoutHandler, JoinPoint joinPoint) throws Throwable {

        Method currentMethod = ((MethodSignature) joinPoint.getSignature()).getMethod();
        Object target = joinPoint.getTarget();
        Method handleMethod = null;
        try {
            handleMethod = joinPoint.getTarget().getClass().getDeclaredMethod(releaseTimeoutHandler, currentMethod.getParameterTypes());
            handleMethod.setAccessible(true);
        } catch (NoSuchMethodException e) {
            throw new IllegalArgumentException("Illegal annotation param customReleaseTimeoutStrategy", e);
        }
        Object[] args = joinPoint.getArgs();

        try {
            handleMethod.invoke(target, args);
        } catch (IllegalAccessException e) {
            throw new CatLockInvocationException("Fail to invoke custom release timeout handler: " + releaseTimeoutHandler, e);
        } catch (InvocationTargetException e) {
            throw e.getTargetException();
        }
    }

    /**
     * 清除当前线程副本
     */
    private void cleanUpThreadLocal() {
        currentThreadLockRes.remove();
        currentThreadLock.remove();
    }


    /**
     * 当前线程锁状态
     */
    @Data
    private class LockRes {

        private LockInfo lockInfo;
        /**
         * 当前锁是否执行释放操作过  true 执行 false 未执行
         */
        private Boolean useState;

        LockRes(LockInfo lockInfo, Boolean useState) {
            this.lockInfo = lockInfo;
            this.useState = useState;
        }
    }
}

```

# 自定义超时策略接口

## 获取锁超时策略

```
/**
 * 获取锁超时的策略接口
 *
 * @author cxc
 * @date 2019年8月8日18:13:34
 **/
public interface LockTimeoutHandler {

    /**
     * 处理
     *
     * @param lockInfo  锁信息
     * @param lock      锁类型
     * @param joinPoint 切面内容
     */
    void handle(LockInfo lockInfo, Lock lock, JoinPoint joinPoint);
}


```



## 释放锁超时策略

```
/**
 * 释放锁超时的处理逻辑接口
 *
 * @author cxc
 * @since 2019年8月8日18:19:18
 **/
public interface ReleaseTimeoutHandler {

    /**
     * 处理
     *
     * @param lockInfo 锁信息
     */
    void handle(LockInfo lockInfo);
}


```

## 自定义加锁超时实现类

```
package com.linjingc.annotationredissonlock.lock.Strategy;

import com.linjingc.annotationredissonlock.lock.basiclock.Lock;
import com.linjingc.annotationredissonlock.lock.exception.CatLockTimeoutException;
import com.linjingc.annotationredissonlock.lock.handler.LockTimeoutHandler;
import com.linjingc.annotationredissonlock.lock.model.LockInfo;
import org.aspectj.lang.JoinPoint;

import java.util.concurrent.TimeUnit;


/**
 * 加锁超时的 策略
 *
 * @author cxc
 * @date 2019年8月8日18:21:28
 **/
public enum LockTimeoutStrategy implements LockTimeoutHandler {

    /**
     * 继续执行业务逻辑，不做任何处理
     */
    NO_OPERATION() {
        @Override
        public void handle(LockInfo lockInfo, Lock lock, JoinPoint joinPoint) {
            // do nothing
        }
    },

    /**
     * 快速失败
     */
    FAIL_FAST() {
        @Override
        public void handle(LockInfo lockInfo, Lock lock, JoinPoint joinPoint) {

            String errorMsg = String.format("Failed to acquire Lock(%s) with timeout(%ds)", lockInfo.getName(), lockInfo.getWaitTime());
            throw new CatLockTimeoutException(errorMsg);
        }
    },

    /**
     * 一直阻塞，直到获得锁，在太多的尝试后，仍会报错
     */
    KEEP_ACQUIRE() {

        private static final long DEFAULT_INTERVAL = 100L;

        private static final long DEFAULT_MAX_INTERVAL = 3 * 60 * 1000L;

        @Override
        public void handle(LockInfo lockInfo, Lock lock, JoinPoint joinPoint) {

            long interval = DEFAULT_INTERVAL;

            while (!lock.acquire()) {

                if (interval > DEFAULT_MAX_INTERVAL) {
                    String errorMsg = String.format("Failed to acquire Lock(%s) after too many times, this may because dead lock occurs.",
                            lockInfo.getName());
                    throw new CatLockTimeoutException(errorMsg);
                }

                try {
                    TimeUnit.MILLISECONDS.sleep(interval);
                    interval <<= 1;
                } catch (InterruptedException e) {
                    throw new CatLockTimeoutException("Failed to acquire Lock", e);
                }
            }
        }
    }
}

```

## 自定义释放锁实现类

```
package com.linjingc.annotationredissonlock.lock.Strategy;

import com.linjingc.annotationredissonlock.lock.exception.CatLockTimeoutException;
import com.linjingc.annotationredissonlock.lock.handler.ReleaseTimeoutHandler;
import com.linjingc.annotationredissonlock.lock.model.LockInfo;

/**
 * 释放超时的 策略
 *
 * @author cxc
 * @date 2019年8月8日18:21:28
 **/
public enum ReleaseTimeoutStrategy implements ReleaseTimeoutHandler {

    /**
     * 继续执行业务逻辑，不做任何处理
     */
    NO_OPERATION() {
        @Override
        public void handle(LockInfo lockInfo) {
            // do nothing
        }
    },
    /**
     * 快速失败
     */
    FAIL_FAST() {
        @Override
        public void handle(LockInfo lockInfo) {

            String errorMsg = String.format("Found Lock(%s) already been released while lock lease time is %d s", lockInfo.getName(), lockInfo.getLeaseTime());
            throw new CatLockTimeoutException(errorMsg);
        }
    }
}


```

# 最后创建自定义异常错误

## 自定义处理锁错误

```
package com.linjingc.annotationredissonlock.lock.exception;

/**
 * 自定义处理锁错误
 *
 * @author cxc
 * @date 2019年8月8日18:16:08
 */
public class CatLockInvocationException extends RuntimeException {

    public CatLockInvocationException() {
    }

    public CatLockInvocationException(String message) {
        super(message);
    }

    public CatLockInvocationException(String message, Throwable cause) {
        super(message, cause);
    }
}


```

## 自定义超时错误

```
package com.linjingc.annotationredissonlock.lock.exception;


/**
 * 自定义锁超时错误
 *
 * @author cxc
 * @date 2019年8月8日18:16:08
 */
public class CatLockTimeoutException extends RuntimeException {

    public CatLockTimeoutException() {
    }

    public CatLockTimeoutException(String message) {
        super(message);
    }

    public CatLockTimeoutException(String message, Throwable cause) {
        super(message, cause);
    }
}


```

这里可以手动捕获一个异常

```
IllegalArgumentException ->参数异常

```

这样基本就构建了一个redisLock注解实现了

# 测试类

```
package com.linjingc.annotationredissonlock.controller;

import com.linjingc.annotationredissonlock.entity.User;
import com.linjingc.annotationredissonlock.lock.Strategy.LockTimeoutStrategy;
import com.linjingc.annotationredissonlock.lock.Strategy.ReleaseTimeoutStrategy;
import com.linjingc.annotationredissonlock.lock.annotation.CatLock;
import com.linjingc.annotationredissonlock.lock.annotation.LockKey;
import lombok.extern.log4j.Log4j2;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import javax.validation.constraints.NotNull;
import java.util.concurrent.TimeUnit;

/**
 * 测试注解
 *
 * @author cxc
 * @date 2019/8/11 21:13
 */

@RestController
@Log4j2
public class TestController {
    @RequestMapping("/")
    public String index() {
        return "测试redis锁注解";
    }


    /**
     * 获取参数 keys param
     *
     * @param param
     * @return
     * @throws Exception
     */
    @RequestMapping("test1")
    @CatLock(waitTime = 10, leaseTime = 60, keys = {"#param"})
    public String getValue(@NotNull String param) throws Exception {
        //  if ("sleep".equals(param)) {//线程休眠或者断点阻塞，达到一直占用锁的测试效果
        Thread.sleep(1000 * 3);
        //}
        return "success";
    }

    /**
     * 默认参数 使用参数keys userId
     *
     * @param userId
     * @param id
     * @return
     * @throws Exception
     */
    @RequestMapping("test2")
    @CatLock(keys = {"#userId"})
    public String getValue(String userId, @LockKey int id) throws Exception {
        Thread.sleep(60 * 1000);
        return "success";
    }

    /**
     * 获取多个参数 keys
     *
     * @param user
     * @return
     * @throws Exception
     */
    @RequestMapping("test3")
    @CatLock(keys = {"#user.name", "#user.id"})
    public String getValue(User user) throws Exception {
        Thread.sleep(60 * 1000);
        return "success";
    }

    /**
     * 测试释放超时
     * leaseTime=-1 表示不超时
     */
    @CatLock(name = "test4", leaseTime = -1, releaseTimeoutStrategy = ReleaseTimeoutStrategy.FAIL_FAST)
    @RequestMapping("test4")
    public String test4() {
        try {
            log.info("foo1 acquire lock");
            TimeUnit.SECONDS.sleep(3);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return "success";

    }

    /**
     * 测试加锁超时 快速失败
     */
    @CatLock(name = "test5", waitTime = 2, lockTimeoutStrategy = LockTimeoutStrategy.FAIL_FAST)
    @RequestMapping("test5")
    public String test5() {
        try {
            log.info("acquire lock");
            TimeUnit.SECONDS.sleep(2);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return "success";

    }

    /**
     * 测试等待超时重试
     */
    @CatLock(name = "test6", waitTime = 2,lockTimeoutStrategy = LockTimeoutStrategy.KEEP_ACQUIRE)
    @RequestMapping("test6")
    public String test6() {
        try {
            TimeUnit.SECONDS.sleep(2);
            log.info("acquire lock");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return "success";

    }

    @CatLock(name = "test7", waitTime = 2, customLockTimeoutStrategy = "customLockTimeout")
    @RequestMapping("test7")
    public String foo4(String foo, String bar) {
        try {
            TimeUnit.SECONDS.sleep(2);
            log.info("acquire lock");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return "success";
    }

    /**
     * 自定义超时处理
     *
     * @param foo
     * @param bar
     * @return
     */
    private String customLockTimeout(String foo, String bar) {
        log.info("customLockTimeout foo: " + foo + " bar: " + bar);
        return "custom foo: " + foo + " bar: " + bar;
    }

}


```

