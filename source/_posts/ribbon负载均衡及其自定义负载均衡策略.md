---
title: ribbon负载均衡及其自定义负载均衡策略
date: 2020-05-09 14:35:18
tags: [SpringCloud,ribbon]
---

# 实现负载均衡

ribbon包中有个IRule接口

## 默认实现好几种负载均衡策略

```
1.RoundRobbionRule  轮训  默认

2.RandomRule 随机

   ->扩展 WeightedResponseTimeRule  响应速度越快的实例选择权重越大,越容易被选择

3.RetryRule    ->先根据轮训策略获取服务      失败后就重试   失败后执行时间重试

4.BestAvailableRule   先过滤多次访问故障而处于短路器跳闸状态的服务,然后选择一个并发量最小的服务

5.AvailabilityFiteringRule   先过滤掉故障实例,再选择并发较小的实例

6.ZoneAvoidanceRule   默认规则,符合判断server所在区域的性能和server的可用性选择服务器
```



继承IRule接口可实现自定义负载均衡策略

修改步骤:

1. 首先注意 不能放在@ComponentScan能扫描的包下 不然就是全局修改负载均衡策略了 
2. 新建一个springboot启动类扫描以外的包 
3. 创建自定义的负载均衡策略 @configuration public class MySelfRule {    @Bean    public IRule myRule(){        return new RandomRule();//自定义策略  ->这边使用配置的策略 随机策略    } } 
4. 启动类上加入注解 @RibbonClient(name="需要配置的服务 比如 helloWord",configuration=myRule.class) 这样就指定了 helloWord服务使用自定义的策略  其他服务还是走默认的

## 轮训算法底层

rest接口第几次请求%集群数量 =实际服务器下标 ,每次重启后 请求次数重置 比如 2台机子 第三次请求  3 %2 会调用第一台机子