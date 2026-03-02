---
title: springboot整合zookeeper实现分布式三curator注解内置锁版本
date: 2019-08-21 21:04:13
tags: [SpringBoot,zookeeper,分布式锁,curator]
---

# springboot整合zookeeper实现分布式三curator注解内置锁版本

[demo地址](https://github.com/AsummerCat/zk-lock/tree/master/annotaion-curator-zk-lock)

这边就利用到curator里面内置的锁了

# 首先还是导入pom

切面 和curator的包

```
  <!-- 切面声明-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-aop</artifactId>
        </dependency>
        <dependency>
            <groupId>org.apache.curator</groupId>
            <artifactId>curator-recipes</artifactId>
            <version>4.2.0</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-configuration-processor</artifactId>
            <optional>true</optional>
        </dependency>
        
```

<!--more-->

# 这边先理解下 curator内置锁类型

基本上都是来自InterProcessLock这个接口的

```
1.可重入锁Shared Reentrant Lock   ->InterProcessMutexLock
2.不可重入锁Shared Lock            ->InterProcessSemaphoreMutex
3.可重入读写锁Shared Reentrant Read Write Lock   ->InterProcessReadWriteLock
4.信号量Shared Semaphore           ->InterProcessSemaphoreV2
5.多锁对象 Multi Shared Lock       ->InterProcessMutex
```



# 创建配置文件

```
server:
  port: 8100

spring:
  application:
    name: annotation-curator-zk-lock


#zk客户端配置重试次数
curator:
  retryCount: 5
  #重试间隔时间
  elapsedTimeMs: 5000
  # zookeeper 地址
  connectString: 127.0.0.1:2181
  # session超时时间
  sessionTimeoutMs: 60000
  # 连接超时时间
  connectionTimeoutMs: 5000
  # 锁节点的根地址
  lockPath: /Lock
  # 锁默认等待时间 (秒)
  lockWaitTime: 15

```

# 创建curator配置类

```
/**
 * zk 初始化配置
 *
 * @author cxc
 * @date 2019年8月13日18:33:28
 */
@Configuration
public class CuratorConfiguration {

    /**
     * 重试次数
     */
    @Value("${curator.retryCount}")
    private int retryCount;
    /**
     * 重试间隔时间
     */
    @Value("${curator.elapsedTimeMs}")
    private int elapsedTimeMs;
    /**
     * zk地址 集群 逗号隔开
     */
    @Value("${curator.connectString}")
    private String connectString;

    /**
     * 设定会话超时时间
     */
    @Value("${curator.sessionTimeoutMs}")
    private int sessionTimeoutMs;
    /**
     * 设定连接时间超时
     */
    @Value("${curator.connectionTimeoutMs}")
    private int connectionTimeoutMs;



    @Bean(initMethod = "start")
    public CuratorFramework curatorFramework() {

        return CuratorFrameworkFactory.builder()
                // 放入zookeeper服务器ip
                .connectString(connectString)
                .sessionTimeoutMs(sessionTimeoutMs)
                .connectionTimeoutMs(connectionTimeoutMs)
                //curator链接zookeeper的策略:ExponentialBackoffRetry
                .retryPolicy(new RetryNTimes(retryCount, elapsedTimeMs))
                .build();

//        return CuratorFrameworkFactory.newClient(
//                connectString,
//                sessionTimeoutMs,
//                connectionTimeoutMs,
//                new RetryNTimes(retryCount, elapsedTimeMs));
    }
}
```



# 创建注解相关的内容

## 创建两个注解

### 方法锁注解

```

/**
 * ZK锁 注解
 *
 * @author cxc
 * @date 2019年8月15日18:09:36
 */
@Target(value = {ElementType.METHOD})
@Retention(value = RetentionPolicy.RUNTIME)
public @interface ZkLock {

    /**
     * 锁的名称
     *
     * @return name
     */
    String name() default "";


    /**
     * 自定义业务key
     *
     * @return keys
     */
    String[] keys() default {};

    /**
     * 锁类型，默认可重入锁
     *
     * @return lockType
     */
    LockType lockType() default LockType.Mutex;

    /**
     * 尝试加锁，最多等待时间(秒)
     *
     * @return waitTime
     */
    long waitTime() default Long.MIN_VALUE;

    /**
     * 加锁超时的处理策略
     *
     * @return lockTimeoutStrategy
     */
    LockTimeoutStrategy lockTimeoutStrategy() default LockTimeoutStrategy.FAIL_FAST;

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
    ReleaseTimeoutStrategy releaseTimeoutStrategy() default ReleaseTimeoutStrategy.FAIL_FAST;

    /**
     * 自定义释放锁时已超时的处理策略
     *
     * @return customReleaseTimeoutStrategy
     */
    String customReleaseTimeoutStrategy() default "";


}


```

### 锁key注解

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

## 创建锁类型枚举

```
package com.linjingc.annotaioncuratorzklock.lock;

public enum LockType {
    /**
     * 可重入锁
     */
    Mutex,
    /**
     * 公平锁 不可重入
     */
    SemaphoreMutex,
    /**
     * 读锁
     */
    Read,
    /**
     * 写锁
     */
    Write,
    /**
     * 联锁
     */
    MultiLock;

    LockType() {
    }

}
```

## 创建基础锁 和锁工厂

### 锁工厂

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
    private CuratorFramework curatorFramework;

    public Lock getLock(LockInfo lockInfo) {
        switch (lockInfo.getType()) {
            case SemaphoreMutex:
                return new InterProcessSemaphoreMutexLock(curatorFramework, lockInfo);
            case Read:
                return new ReadLock(curatorFramework, lockInfo);
            case Write:
                return new WriteLock(curatorFramework, lockInfo);
            default:
                return new InterProcessMutexLock(curatorFramework, lockInfo);
        }
    }

}

```

### 基础锁接口 实现加锁 和释放锁

```
package com.linjingc.annotaioncuratorzklock.lock.basiclock;

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

### 然后开始创建几个基础锁类型

### 可重入锁

```
/**
 * 可重入锁
 *
 * @author cxc
 * @date 2019年8月19日17:10:22
 */
@Data
@Log4j2
public class InterProcessMutexLock implements Lock {

    private InterProcessLock interProcessLock;

    private final LockInfo lockInfo;

    private CuratorFramework curatorFramework;

    public InterProcessMutexLock(CuratorFramework curatorFramework, LockInfo info) {
        this.curatorFramework = curatorFramework;
        this.lockInfo = info;
    }


    @Override
    public boolean acquire() {
        interProcessLock = new InterProcessMutex(curatorFramework, lockInfo.getLockPath());
        try {
            boolean acquire = interProcessLock.acquire(lockInfo.getWaitTime(), TimeUnit.SECONDS);
            if (acquire) {
                return true;
            }
        } catch (Exception e) {
            log.error("zk加锁错误", e);
            return false;
        }
        return false;
    }

    @Override
    public boolean release() {
        boolean acquiredInThisProcess = interProcessLock.isAcquiredInThisProcess();
        if (acquiredInThisProcess) {
            try {
                interProcessLock.release();
                System.out.println("解锁成功");
                return true;
            } catch (Exception e) {
                log.error("zk解锁错误", e);
                return false;
            }
        }
        return false;
    }
}


```

### 不可重入锁

```
/**
 * 不可重入锁
 *
 * @author cxc
 * @date 2019年8月19日17:10:22
 */
@Data
@Log4j2
public class InterProcessSemaphoreMutexLock implements Lock {

    private InterProcessLock interProcessLock;

    private final LockInfo lockInfo;

    private CuratorFramework curatorFramework;

    public InterProcessSemaphoreMutexLock(CuratorFramework curatorFramework, LockInfo info) {
        this.curatorFramework = curatorFramework;
        this.lockInfo = info;
    }


    @Override
    public boolean acquire() {
        interProcessLock = new InterProcessSemaphoreMutex(curatorFramework, lockInfo.getLockPath());
        try {
            boolean acquire = interProcessLock.acquire(lockInfo.getWaitTime(), TimeUnit.SECONDS);
            if (acquire) {
                return true;
            }
        } catch (Exception e) {
            log.error("zk加锁错误", e);
            return false;
        }
        return false;
    }

    @Override
    public boolean release() {
        boolean acquiredInThisProcess = interProcessLock.isAcquiredInThisProcess();
        if (acquiredInThisProcess) {
            try {
                interProcessLock.release();
                System.out.println("解锁成功");
                return true;
            } catch (Exception e) {
                log.error("zk解锁错误", e);
                return false;
            }
        }
        return false;
    }
}


```

### 读写锁 读锁

```
/**
 * 读写锁 读锁
 *
 * @author cxc
 * @date 2019年8月19日17:10:22
 */
@Data
@Log4j2
public class ReadLock implements Lock {

    private InterProcessReadWriteLock interProcessLock;

    private final LockInfo lockInfo;

    private CuratorFramework curatorFramework;

    public ReadLock(CuratorFramework curatorFramework, LockInfo info) {
        this.curatorFramework = curatorFramework;
        this.lockInfo = info;
    }


    @Override
    public boolean acquire() {
        interProcessLock = new InterProcessReadWriteLock(curatorFramework, lockInfo.getLockPath());
        try {
            boolean acquire = interProcessLock.readLock().acquire(lockInfo.getWaitTime(), TimeUnit.SECONDS);
            if (acquire) {
                return true;
            }
        } catch (Exception e) {
            log.error("zk加锁错误", e);
            return false;
        }
        return false;
    }

    @Override
    public boolean release() {
        boolean acquiredInThisProcess = interProcessLock.readLock().isAcquiredInThisProcess();
        if (acquiredInThisProcess) {
            try {
                interProcessLock.readLock().release();
                System.out.println("解锁成功");
                return true;
            } catch (Exception e) {
                log.error("zk解锁错误", e);
                return false;
            }
        }
        return false;
    }
}


```

### 读写锁 写锁

```
/**
 * 读写锁 写锁
 *
 * @author cxc
 * @date 2019年8月19日17:10:22
 */
@Data
@Log4j2
public class WriteLock implements Lock {

    private InterProcessReadWriteLock interProcessLock;

    private final LockInfo lockInfo;

    private CuratorFramework curatorFramework;

    public WriteLock(CuratorFramework curatorFramework, LockInfo info) {
        this.curatorFramework = curatorFramework;
        this.lockInfo = info;
    }


    @Override
    public boolean acquire() {
        interProcessLock = new InterProcessReadWriteLock(curatorFramework, lockInfo.getLockPath());
        try {
            boolean acquire = interProcessLock.writeLock().acquire(lockInfo.getWaitTime(), TimeUnit.SECONDS);
            if (acquire) {
                return true;
            }
        } catch (Exception e) {
            log.error("zk加锁错误", e);
            return false;
        }
        return false;
    }

    @Override
    public boolean release() {
        boolean acquiredInThisProcess = interProcessLock.writeLock().isAcquiredInThisProcess();
        if (acquiredInThisProcess) {
            try {
                interProcessLock.writeLock().release();
                System.out.println("解锁成功");
                return true;
            } catch (Exception e) {
                log.error("zk解锁错误", e);
                return false;
            }
        }
        return false;
    }
}


```

## 创建异常处理的异常类 

### 自定义处理锁错误

```
package com.linjingc.annotaioncuratorzklock.lock.exception;

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

### 自定义锁超时错误

```
package com.linjingc.annotaioncuratorzklock.lock.exception;


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

## 创建两个策略接口 用来实现 加锁失败 和解锁超时的处理

# 创建锁切面类

```
/**
 * zk锁切面类
 * 用来包裹方法使用
 *
 * @author cxc
 * @date 2019年8月8日17:19:55
 */
@Aspect
@Component
@Order(0)
@Log4j2
public class ZkLockAspectAop {
    @Autowired
    private LockInfoProvider lockInfoProvider;
    @Autowired
    private LockFactory lockFactory;

    /**
     * 当前锁
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
     * @param zkLock    锁类型
     * @return
     * @throws Throwable
     */
    @Around(value = "@annotation(zkLock)")
    public Object around(ProceedingJoinPoint joinPoint, ZkLock zkLock) throws Throwable {
        //获取自定义锁信息
        LockInfo lockInfo = lockInfoProvider.get(joinPoint, zkLock);
        Lock lock = lockFactory.getLock(lockInfo);
        //设置当前锁状态
        currentThreadLockRes.set(new LockRes(lockInfo, false));
        //加锁
        boolean carryLock = lock.acquire();
        if (!carryLock) {
            if (log.isWarnEnabled()) {
                log.warn("Timeout while acquiring Lock({})", lockInfo.getNodePath());
            }
            //如果有自定义的超时策略
            if (!StringUtils.isEmpty(zkLock.customLockTimeoutStrategy())) {
                return handleCustomLockTimeout(zkLock.customLockTimeoutStrategy(), joinPoint);
            } else {
                //否则使用配置的超时处理策略
                zkLock.lockTimeoutStrategy().handle(lockInfo, lock, joinPoint);
            }
        }
        System.out.println("加锁成功");


        currentThreadLock.set(lock);


        return joinPoint.proceed();
    }


    /**
     * 方法执行完毕 释放锁
     *
     * @param joinPoint
     * @param
     * @throws Throwable
     */
    @AfterReturning(value = "@annotation(zkLock)")
    public void afterReturning(JoinPoint joinPoint, ZkLock zkLock) throws Throwable {
        //释放锁
        releaseLock(zkLock, joinPoint);
        //清理线程副本
        cleanUpThreadLocal();
    }

    /**
     * 切面 异常处理
     *
     * @param joinPoint
     * @param
     * @param ex
     * @throws Throwable
     */
    @AfterThrowing(value = "@annotation(zkLock)", throwing = "ex")
    public void afterThrowing(JoinPoint joinPoint, ZkLock zkLock, Throwable ex) throws Throwable {
        //释放锁
        releaseLock(zkLock, joinPoint);
        //清理线程副本
        cleanUpThreadLocal();
        throw ex;
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

    /**
     * 清除当前线程副本
     */
    private void cleanUpThreadLocal() {
        currentThreadLockRes.remove();
        currentThreadLock.remove();
    }

    /**
     * 释放锁 避免重复释放锁
     * 如: 执行完毕释放一次 throw时又释放一次
     */
    private void releaseLock(ZkLock zkLock, JoinPoint joinPoint) throws Throwable {
        LockRes lockRes = currentThreadLockRes.get();
        //未执行过释放锁操作
        if (!lockRes.getUseState()) {
            boolean releaseRes = currentThreadLock.get().release();
            // avoid release lock twice when exception happens below
            lockRes.setUseState(true);
            if (!releaseRes) {
                handleReleaseTimeout(zkLock, lockRes.getLockInfo(), joinPoint);
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
    private void handleReleaseTimeout(ZkLock zkLock, LockInfo lockInfo, JoinPoint joinPoint) throws Throwable {

        if (log.isWarnEnabled()) {
            log.warn("Timeout while release Lock({})", lockInfo.getNode());
        }

        if (!StringUtils.isEmpty(zkLock.customReleaseTimeoutStrategy())) {

            handleCustomReleaseTimeout(zkLock.customReleaseTimeoutStrategy(), joinPoint);

        } else {
            zkLock.releaseTimeoutStrategy().handle(lockInfo);
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

}


```

# 创建生成锁key的节点方法

```
/**
 * 获取用户定义业务key
 *
 * @author cxc
 * @date 2019年08月08日20:52:40
 */
@Component
public class BusinessKeyProvider {

    public static final String LOCK_NAME_SEPARATOR = "/";

    private ParameterNameDiscoverer nameDiscoverer = new DefaultParameterNameDiscoverer();

    private ExpressionParser parser = new SpelExpressionParser();

    public String getKeyName(ProceedingJoinPoint joinPoint, ZkLock zkLock) {
        List<String> keyList = new ArrayList<>();
        Method method = getMethod(joinPoint);
        //获取方法zkLock注解上的自定义keys
        List<String> definitionKeys = getSpelDefinitionKey(zkLock.keys(), method, joinPoint.getArgs());
        keyList.addAll(definitionKeys);
        //获取参数注解LockKey 上的内容
        List<String> parameterKeys = getParameterKey(method.getParameters(), joinPoint.getArgs());
        keyList.addAll(parameterKeys);
        //进行拼接
        return StringUtils.collectionToDelimitedString(keyList, "", LOCK_NAME_SEPARATOR, "");
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

# 创建锁提供者 这边生成锁实例

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
     * 锁的key根路径
     */

    @Value("${curator.lockPath}")
    private String LOCK_NAME_PREFIX;
    public static final String LOCK_NAME_SEPARATOR = "/";
    /**
     * 加锁等待时间
     */
    @Value("${curator.lockWaitTime}")
    private Long lockWaitTime;


    /**
     * 自定义业务key
     */
    @Autowired
    private BusinessKeyProvider businessKeyProvider;


    /***
     * 获取锁信息
     * 锁的名称 = 根路径+子路径+锁名
     * @param joinPoint
     * @param zkLock
     * @return
     */
    public LockInfo get(ProceedingJoinPoint joinPoint, ZkLock zkLock) {
        //获取到锁类型
        LockType type = zkLock.lockType();
        //获取到切面的信息
        MethodSignature signature = (MethodSignature) joinPoint.getSignature();
        //根据自定义业务key 获取keyName
        String businessKeyName = businessKeyProvider.getKeyName(joinPoint, zkLock);
        //拼接lockName地址
        String lockPath = LOCK_NAME_PREFIX + LOCK_NAME_SEPARATOR + getName(zkLock.name(), signature) + businessKeyName;

        //获取等待时间 不设置则根据配置的lockWaitTime的生成
        long waitTime = getWaitTime(zkLock);
        //实例化锁
        return new LockInfo(lockPath,type,waitTime);
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
    private long getWaitTime(ZkLock lock) {
        return lock.waitTime() == Long.MIN_VALUE ?
                lockWaitTime : lock.waitTime();
    }

}


```



# 原生测试方法

```
package com.linjingc.annotaioncuratorzklock.controller;

import org.apache.curator.framework.CuratorFramework;
import org.apache.curator.framework.recipes.locks.InterProcessMutex;
import org.apache.curator.framework.recipes.locks.InterProcessReadWriteLock;
import org.apache.curator.framework.recipes.locks.InterProcessSemaphoreMutex;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.concurrent.TimeUnit;

/**
 * 基础锁测试
 *
 * @author cxc
 * @date 2019年8月20日18:10:26
 */
@RestController
public class TestController {

    @Autowired
    private CuratorFramework curatorFramework;


    /**
     * 写锁
     * 只能存在一个
     *
     * @return
     */
    @RequestMapping("test")
    public String test() {
        InterProcessReadWriteLock interProcessReadWriteLock = new InterProcessReadWriteLock(curatorFramework, "/lock/test");
        try {
            boolean acquire = interProcessReadWriteLock.writeLock().acquire(4, TimeUnit.SECONDS);
            boolean acquire1 = interProcessReadWriteLock.writeLock().acquire(4, TimeUnit.SECONDS);
            if (acquire) {
                System.out.println("加锁成功");
            }
            if (acquire1) {
                System.out.println("加锁成功1");
            }

            interProcessReadWriteLock.writeLock().release();
            interProcessReadWriteLock.writeLock().release();
        } catch (Exception e) {
            e.printStackTrace();
        }
        return "success";
    }


    /**
     * 读锁
     * 能存在多个
     *
     * @return
     */
    @RequestMapping("test1")
    public String test1() {
        InterProcessReadWriteLock interProcessReadWriteLock = new InterProcessReadWriteLock(curatorFramework, "/lock/test");
        try {
            boolean acquire = interProcessReadWriteLock.readLock().acquire(4, TimeUnit.SECONDS);
            boolean acquire1 = interProcessReadWriteLock.readLock().acquire(4, TimeUnit.SECONDS);
            if (acquire) {
                System.out.println("加锁成功");
            }
            if (acquire1) {
                System.out.println("加锁成功1");
            }

            interProcessReadWriteLock.readLock().release();
            interProcessReadWriteLock.readLock().release();
        } catch (Exception e) {
            e.printStackTrace();
        }
        return "success";
    }

    /**
     * 可重入锁
     * 能存在多个 但是必须都解锁
     *
     * @return
     */
    @RequestMapping("test2")
    public String test2() {
        InterProcessMutex interProcessMutex = new InterProcessMutex(curatorFramework, "/lock/test");
        try {
            boolean acquire = interProcessMutex.acquire(4, TimeUnit.SECONDS);
            boolean acquire1 = interProcessMutex.acquire(4, TimeUnit.SECONDS);
            if (acquire) {
                System.out.println("加锁成功");
            }
            if (acquire1) {
                System.out.println("加锁成功1");
            }

            interProcessMutex.release();
            interProcessMutex.release();
        } catch (Exception e) {
            e.printStackTrace();
        }
        return "success";
    }

    /**
     * 不可重入锁
     * 只能存在一个
     *
     * @return
     */
    @RequestMapping("test3")
    public String test3() {
        InterProcessSemaphoreMutex interProcessSemaphoreMutex = new InterProcessSemaphoreMutex(curatorFramework, "/lock/test");
        try {
            boolean acquire = interProcessSemaphoreMutex.acquire(4, TimeUnit.SECONDS);
            if (acquire) {
                System.out.println("加锁成功");
            }
            interProcessSemaphoreMutex.release();
        } catch (Exception e) {
            e.printStackTrace();
        }
        return "success";
    }
}



```



# 集成注解测试方法

```
package com.linjingc.annotaioncuratorzklock.controller;

import com.linjingc.annotaioncuratorzklock.lock.LockType;
import com.linjingc.annotaioncuratorzklock.lock.annotaion.ZkLock;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

/**
 * 注解测试
 *
 * @author cxc
 * @date 2019年8月20日18:10:08
 */
@RestController
public class LockController {

    //todo 手动捕获这个错误  IllegalMonitorStateException

    /**
     * 可重入锁
     *
     * @return
     */
    @ZkLock(lockType = LockType.Mutex)
    @RequestMapping("locktest")
    public String test() {
        return "success";
    }

    /**
     * 不可重入锁
     *
     * @param test
     * @return
     */
    @ZkLock(lockType = LockType.SemaphoreMutex)
    @RequestMapping("locktest1")
    public String test1(String test) {
        return "success";
    }

    /**
     * 读锁
     *
     * @param test
     * @return
     */
    @ZkLock(lockType = LockType.Read)
    @RequestMapping("locktest2")
    public String test2(String test) {
        return "success";
    }

    /**
     * 写锁
     *
     * @param test
     * @return
     */
    @ZkLock(lockType = LockType.Write)
    @RequestMapping("locktest3")
    public String test3(String test) {
        return "success";
    }
}


```

