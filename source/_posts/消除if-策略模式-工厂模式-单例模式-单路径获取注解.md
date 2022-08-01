---
title: 消除if-策略模式+工厂模式+单例模式-单路径获取注解
date: 2019-08-20 20:04:51
tags: [java,设计模式]
---

# 消除if-策略模式+工厂模式+单例模式-单路径获取注解

这样处理的好处是 : 如果有多个if 判断执行多个分支 后期修改代码比较麻烦

使用注解 标记这个分支 使用工厂 自动匹配这个分支执行

以后就只要维护注解 和具体分支就可以了 不用改动其他的 

# 导入pom

```
<!-- 操作反射 -->
   <dependency>
      <groupId>org.reflections</groupId>
      <artifactId>reflections</artifactId>
      <version>0.9.10</version>
   </dependency>
  
```

<!--more-->

# 创建注解

```
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface PayChannel {
    int channelId();
}

```

# 创建工厂类

- 单例模式获取 工厂实例
- 扫描指定实现类，存储所有的注解信息->及其匹配的class

```
public class PayStrategyFactory {
    //饿汉式 单例模式
    private static PayStrategyFactory payStrategyFactory=new PayStrategyFactory();

    /**
     * 单例模式-私有构造器
     */
    private PayStrategyFactory(){

    }
    public static PayStrategyFactory getInstance(){
        return payStrategyFactory;
    }

    /**
     * 重点：存储所有的payChannel
     */
    public static HashMap<Integer,String> payChannelMap=new HashMap<>();

    static {
        //1、扫描支付渠道的实现类 ,Reflections 依赖 Google 的 Guava 库和 Javassist 库
        Reflections reflections = new Reflections("com.yf.custom.pay.impl");
        //2、获取所有包含PayChannel注解的类
        Set<Class<?>> classList = reflections.getTypesAnnotatedWith(PayChannel.class);
        for (Class clazz : classList) {
            PayChannel t = (PayChannel) clazz.getAnnotation(PayChannel.class);
            //3、赋值payChannelMap，存储所有的支付渠道
            payChannelMap.put(t.channelId(), clazz.getCanonicalName());
        }

    }
    /**
     *  根据channelId获取对应的具体实现
     * @param channelId
     * @return
     */
    public PayStrategy create(Integer channelId) throws ClassNotFoundException, IllegalAccessException, InstantiationException {
        //1.获取渠道对应的类名
        String clazz=payChannelMap.get(channelId);
        Class clazz_=Class.forName(clazz);
        /**
         * newInstance ：工厂模式经常使用newInstance来创建对象，newInstance()是实现IOC、反射、依赖倒置 等技术方法的必然选择
         *      调用class的加载方法加载某个类，然后实例化
         *
         * new 只能实现具体类的实例化，不适合于接口编程
         */
        return (PayStrategy) clazz_.newInstance();
    }
}
```

# 创建策略接口及其实现

## 定义价格接口PayStrategy

```
public interface PayStrategy {
    /**
     * 根据渠道计算价格
     * @param channelId
     * @param goodsId
     * @return
     */
    BigDecimal calRecharge(Integer channelId,Integer goodsId);
}
```

## 创建实现类 及其标记注解

```
@PayChannel(channelId = 1)
public class ICBCBank implements PayStrategy{
    @Override
    public BigDecimal calRecharge(Integer channelId, Integer goodsId) {
        return new BigDecimal("1");
    }
}
```

```
@PayChannel(channelId = 2)
public class CMBCBank implements PayStrategy{
    @Override
    public BigDecimal calRecharge(Integer channelId, Integer goodsId) {
        return new BigDecimal("2");
    }
}
```

有需要的话继续扩展，创建实现类即可

# 创建上下文

```
public class PayChannelContext {
    /**
     * 支付渠道上下文：策略模式，根据channelId动态获取对应实现类并执行
     * @param channelId
     * @param goodsId
     * @return
     * @throws Exception
     */
    BigDecimal calRecharge(Integer channelId, Integer goodsId) throws Exception {
        //1.获取工厂
        PayStrategyFactory payStrategyFactory=PayStrategyFactory.getInstance();
        //2.调用具体实现
        PayStrategy payStrategy=payStrategyFactory.create(channelId);
        //3.执行并返回结果
        return payStrategy.calRecharge(channelId,goodsId);
    }
}
```



# 执行方法

```
 @RequestMapping("calculatePrice")
    public String calculatePrice(@RequestParam  Integer channelId, @RequestParam Integer goodsId) throws Exception {
        PayChannelContext context=new PayChannelContext();
        BigDecimal bigDecimal= context.calRecharge(channelId,goodsId);
        return bigDecimal.setScale(2)+"";
    }
```



