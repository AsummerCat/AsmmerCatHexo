---
title: Hystrix监控面板dashboard单机
date: 2019-01-14 09:41:09
tags: [springCloud,Hystrix,Dashboard]
---

# 监控台

接口实现使用了`@HystrixCommand`修饰，所以这个接口的调用情况会被Hystrix记录下来，以用来给断路器和Hystrix Dashboard使用。

<!--more-->

# 添加相关dashboard的pom

```java
<dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-hystrix</artifactId>
    </dependency>
        
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-hystrix-dashboard</artifactId>
    </dependency>
    
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
```

# 在启动类添加开启监控台的注解

- 为应用主类加上`@EnableHystrixDashboard`，启用Hystrix Dashboard功能。

```java
package com.linjing.hystrixdashboard;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.SpringCloudApplication;
import org.springframework.cloud.netflix.hystrix.dashboard.EnableHystrixDashboard;

//@SpringBootApplication
@EnableHystrixDashboard
@SpringCloudApplication
public class HystrixDashboardApplication {

    public static void main(String[] args) {
        SpringApplication.run(HystrixDashboardApplication.class, args);
    }
}
```

 

到这里我们已经完成了基本配置，接下来我们可以启动该应用，并访问：`http://localhost:8084/hystrix`

这里的端口号指的是 你项目的端口号

![图片](/img/2019-1-14/HystrixDashboard.png)

这是Hystrix Dashboard的监控首页，该页面中并没有具体的监控信息。从页面的文字内容中我们可以知道，Hystrix Dashboard共支持三种不同的监控方式，依次为：

- 默认的集群监控：通过URL`http://turbine-hostname:port/turbine.stream`开启，实现对默认集群的监控。
- 指定的集群监控：通过URL`http://turbine-hostname:port/turbine.stream?cluster=[clusterName]`开启，实现对clusterName集群的监控。
- 单体应用的监控：通过URL`http://hystrix-app:port/hystrix.stream`开启，实现对具体某个服务实例的监控。

前两者都对集群的监控，需要整合Turbine才能实现，

# 开启监控的前提

既然Hystrix Dashboard监控单实例节点需要通过访问实例的`/hystrix.stream`接口来实现，自然我们需要为服务实例添加这个端点，而添加该功能的步骤也同样简单，只需要下面两步：

- 在服务实例`pom.xml`中的`dependencies`节点中新增`spring-boot-starter-actuator`监控模块以开启监控相关的端点，并确保已经引入断路器的依赖`spring-cloud-starter-hystrix`：

```java
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-hystrix</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

- 确保在服务实例的主类中已经使用`@EnableCircuitBreaker`或`@EnableHystrix`注解，开启了断路器功能。

到这里已经完成了所有的配置，我们可以在Hystrix Dashboard的首页输入`http://localhost:9001/hystrix.stream`，已启动对“hystrixConsumer”的监控，点击“Monitor Stream”按钮，此时我们可以看到如下页面：

![](/img/2019-1-14/HystrixDashboard1.png)

在对该页面介绍前，我们先看看在首页中我们还没有介绍的两外两个参数：

- `Delay`：该参数用来控制服务器上轮询监控信息的延迟时间，默认为2000毫秒，我们可以通过配置该属性来降低客户端的网络和CPU消耗。
- `Title`：该参数对应了上图头部标题Hystrix Stream之后的内容，默认会使用具体监控实例的URL，我们可以通过配置该信息来展示更合适的标题。

回到监控页面，我们来详细说说其中各元素的具体含义：

- 我们可以在监控信息的左上部分找到两个重要的图形信息：一个实心圆和一条曲线。
  - 实心圆：共有两种含义。它通过颜色的变化代表了实例的健康程度，如下图所示，它的健康度从绿色、黄色、橙色、红色递减。该实心圆除了颜色的变化之外，它的大小也会根据实例的请求流量发生变化，流量越大该实心圆就越大。所以通过该实心圆的展示，我们就可以在大量的实例中快速的发现故障实例和高压力实例。

![](/img/2019-1-14/HystrixDashboard2.png)

- 曲线：用来记录2分钟内流量的相对变化，我们可以通过它来观察到流量的上升和下降趋势。

其他一些数量指标如下图所示：

![](/img/2019-1-14/HystrixDashboard3.png)