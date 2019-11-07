---
title: springboot整合zookeeper实现分布式二curator注解版本
date: 2019-08-21 21:02:47
tags: [SpringBoot,zookeeper,分布式锁,curator]
---

# springboot整合zookeeper实现分布式二curator注解版本

[demo地址](https://github.com/AsummerCat/zk-lock/tree/master/annotation-zk-lock)

# 导入pom

```java
  <dependency>
            <groupId>org.apache.curator</groupId>
            <artifactId>curator-recipes</artifactId>
            <version>4.2.0</version>
        </dependency>
        <!-- 切面声明-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-aop</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-configuration-processor</artifactId>
            <optional>true</optional>
        </dependency>
```

<!--more-->

# 创建配置文件

```java
server:
  port: 8090

spring:
  application:
    name: annotation-zk-lock


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
  lockPath: /Lock

```

# 创建curator 配置文件

生成zk客户端

```java
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



# 添加锁注解

## 锁注解

### 方法锁

```java
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


}

```

### 锁key 用来标记

```java
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

## 创建锁实体

```java
/**
 * 锁基本 信息
 *
 * @author cxc
 * @date 2019年8月8日17:13:18
 */
@Data
public class LockInfo {
    /**
     * 临时目录分隔符
     */
    public final static String SEPARATOR_CHARACTER="/";

    /**
     * 当前锁节点
     */
    private String node;

    /**
     * 当前锁 前一节点
     */
    private String lastNode;


    /**
     * 当前锁的索引路径
     */
    private String lockPath;


    public LockInfo() {
    }

    public LockInfo(String lockPath) {
        this.lockPath = lockPath;
    }


    /**
     * 获取当前节点位置路径
     * @return
     */
    public String getNodePath(){
        return lockPath+SEPARATOR_CHARACTER+node;
    }
    /**
     * 获取当前前一个节点位置路径
     * @return
     */
    public String getLastNodePath(){
        return lockPath+SEPARATOR_CHARACTER+lastNode;
    }

}

```

## 创建锁切面类

```java
/**
 * zk锁切面类
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
public class ZkLockAspectAop {
    @Autowired
    private LockInfoProvider lockInfoProvider;
    @Autowired
    private CuratorFramework zkClient;

    /**
     * 当前锁
     */
    ThreadLocal<LockInfo> currentThreadLock = new ThreadLocal<>();



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
        zkLock.name();
        zkLock.keys();
        //获取自定义锁信息
        LockInfo lockInfo = lockInfoProvider.get(joinPoint, zkLock);
        try {
            //根节点的初始化放在构造函数里面不生效
            if (zkClient.checkExists().forPath(lockInfo.getLockPath()) == null) {
                System.out.println("初始化根节点==========>" + lockInfo.getLockPath());
                zkClient.create().creatingParentsIfNeeded().forPath(lockInfo.getLockPath());
                System.out.println("当前线程" + Thread.currentThread().getName() + "初始化根节点" + lockInfo.getLockPath());
            }
        } catch (Exception e) {
            throw new RuntimeException("构建根节点失败");
        }
        try {
            //创建临时节点
            lockInfo.setNode(zkClient.create().withMode(CreateMode.EPHEMERAL_SEQUENTIAL).forPath(lockInfo.getLockPath() + "/").replace(lockInfo.getLockPath()+LockInfo.SEPARATOR_CHARACTER,""));
            currentThreadLock.set(lockInfo);
        } catch (Exception e) {
            throw new RuntimeException("zk创建锁节点失败");
        }
        //检查是否获取成功锁 不成功阻塞线程
        checkLock();

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
        unlock();
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
        unlock();
        throw ex;
    }



    /**
     * 检查是否获取锁
     * 检查是否获取成功锁 不成功阻塞线程
     */
    private void checkLock() {
        //判断获取锁
        if (!tryLock()) {
            //监听
            waiForLock();
            //前锁解除继续判断获取
            checkLock();
        }
    }

    public void unlock() {
        try {
            zkClient.delete().guaranteed().deletingChildrenIfNeeded().forPath(currentThreadLock.get().getNodePath());
            System.out.println(currentThreadLock.get().getNodePath() + "解锁成功");
            currentThreadLock.remove();
        } catch (Exception e) {
            //guaranteed()保障机制，若未删除成功，只要会话有效会在后台一直尝试删除
        }
    }

    /**
     * 阻塞监听节点  锁等待
     */
    private void waiForLock() {
        CountDownLatch cdl = new CountDownLatch(1);
        //创建监听器watch
        NodeCache nodeCache = new NodeCache(zkClient,currentThreadLock.get().getLastNodePath());
        try {
            nodeCache.start(true);
            nodeCache.getListenable().addListener(new NodeCacheListener() {

                @Override
                public void nodeChanged() throws Exception {
                    System.out.println(currentThreadLock.get().getLastNodePath() + "节点监听事件触发");
                    cdl.countDown();
                }
            });
        } catch (Exception e) {
        }
        //如果前一个节点还存在，则阻塞自己
        try {
            if (zkClient.checkExists().forPath(currentThreadLock.get().getLastNode()) != null) {
                cdl.await();
            }
        } catch (Exception e) {
        } finally {
            //阻塞结束，说明自己是最小的节点，则取消watch，开始获取锁
            try {
                nodeCache.close();
            } catch (IOException e) {
            }
        }
    }

    /**
     * 获取锁
     *
     * @return
     */
    private boolean tryLock() {
        try {
            List<String> childrens = this.zkClient.getChildren().forPath(currentThreadLock.get().getLockPath());
            Collections.sort(childrens);
            if (currentThreadLock.get().getNode().equals(childrens.get(0))) {
                System.out.println("当前线程获得锁" + currentThreadLock.get().getNodePath());
                return true;
            } else {
                //取前一个节点
                int curIndex = childrens.indexOf(currentThreadLock.get().getNodePath().substring(currentThreadLock.get().getLockPath().length() + 1));
                //如果是-1表示children里面没有该节点
                currentThreadLock.get().setLastNode(childrens.get(curIndex - 1));
                System.out.println("前一个节点:" + childrens.get(curIndex - 1));
            }
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
        return false;
    }
}

```

## 创建锁提供者

```java
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
        //获取到切面的信息
        MethodSignature signature = (MethodSignature) joinPoint.getSignature();
        //根据自定义业务key 获取keyName
        String businessKeyName = businessKeyProvider.getKeyName(joinPoint, zkLock);
        //拼接lockName地址
        String lockPath = LOCK_NAME_PREFIX + LOCK_NAME_SEPARATOR + getName(zkLock.name(), signature) + businessKeyName;
        //实例化锁
        return new LockInfo(lockPath);
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


}

```

## 创建 获取keyname的方法

```java
package com.linjingc.annotationzklock.lock.core;

import com.linjingc.annotationzklock.lock.annotaion.LockKey;
import com.linjingc.annotationzklock.lock.annotaion.ZkLock;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.reflect.MethodSignature;
import org.springframework.context.expression.MethodBasedEvaluationContext;
import org.springframework.core.DefaultParameterNameDiscoverer;
import org.springframework.core.ParameterNameDiscoverer;
import org.springframework.expression.EvaluationContext;
import org.springframework.expression.ExpressionParser;
import org.springframework.expression.spel.standard.SpelExpressionParser;
import org.springframework.expression.spel.support.StandardEvaluationContext;
import org.springframework.stereotype.Component;
import org.springframework.util.StringUtils;

import java.lang.reflect.Method;
import java.lang.reflect.Parameter;
import java.util.ArrayList;
import java.util.List;

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



这样基本就构建完成了

生成的key节点为 方法名全路径 + lockkey

# 测试方法

```java
@RestController
public class TestController {

    /**
     * 默认使用全路径
     * @return
     */
    @ZkLock()
    @RequestMapping("test")
    public String test(){
        return "success";
    }

    @ZkLock()
    @RequestMapping("test1")
    public String test1(@LockKey String test){
        return "success";
    }
}


```

