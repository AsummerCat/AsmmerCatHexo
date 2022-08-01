---
title: Bean的作用域
date: 2018-11-21 16:31:50
tags: [Spring]
---


# Bean的作用域

当开发者定义Bean的时候，同时也会定义了该如何创建Bean实例。这些具体创建的过程是很重要的，因为只有通过对这些过程的配置，开发者才能创建实例对象。

开发者不仅可以控制注入不同的依赖到Bean之中，也可以配置Bean的作用域。这种方法是非常强大而且弹性也非常好的。开发者可以通过配置来指定对象的作用域，而不用在Java类层次上来配置。Bean可以配置多种作用域。 Spring框架支持5种作用域，有三种作用域是当开发者使用基于web的ApplicationContext的时候才生效的。 

 
下面就是Spring直接支持的作用域了，当然开发者也可以自己定制作用域。

<!--more-->

<table>
<thead>
<tr>
<th>作用域</th>
<th>描述</th>
</tr>
</thead>
<tbody>
<tr>
<td>单例(singleton)</td>
<td>（默认）每一个Spring IoC容器都拥有唯一的一个实例对象</td>
</tr>
<tr>
<td>原型（prototype）</td>
<td>一个Bean定义可以创建任意多个实例对象</td>
</tr>
<tr>
<td>请求（request）</td>
<td>一个HTTP请求会产生一个Bean对象，也就是说，每一个HTTP请求都有自己的Bean实例。只在基于web的Spring <code>ApplicationContext</code>中可用</td>
</tr>
<tr>
<td>会话（session）</td>
<td>限定一个Bean的作用域为HTTP<code>session</code>的生命周期。同样，只有基于web的Spring <code>ApplicationContext</code>才能使用</td>
</tr>
<tr>
<td>全局会话（global session）</td>
<td>限定一个Bean的作用域为全局HTTP<code>Session</code>的生命周期。通常用于门户网站场景，同样，只有基于web的Spring <code>ApplicationContext</code>可用</td>
</tr>
<tr>
<td>应用(application)</td>
<td>限定一个Bean的作用域为<code>ServletContext</code>的生命周期。同样，只有基于web的Spring <code>ApplicationContext</code>可用</td>
</tr>
</tbody>
</table>


# The singleton scope

单例Bean全局只有一个共享的实例，所有将单例Bean作为依赖的情况下，容器返回将是同一个实例。
换言之，当开发者定义一个Bean的作用域为单例时，Spring IoC容器只会根据Bean定义来创建该Bean的唯一实例。这些唯一的实例会缓存到容器中，后续针对单例Bean的请求和引用，都会从这个缓存中拿到这个唯一的实例。 

![单例](/img/2018-11-21/singleton.png)

Spring的单例Bean和与设计模式之中的所定义的单例模式是有所区别的。设计模式中的单例模式是将一个对象的作用域硬编码的，一个ClassLoader只有唯一的一个实例。 而Spring的单例作用域，是基于每个容器，每个Bean只有一个实例。这意味着，如果开发者根据一个类定义了一个Bean在单个的Spring容器中，那么Spring容器会根据Bean定义创建一个唯一的Bean实例。   

* 单例作用域是Spring的默认作用域，下面的例子是在基于XML的配置中配置单例模式的Bean。

```
<bean id="accountService" class="com.foo.DefaultAccountService"/>

<!-- the following is equivalent, though redundant (singleton scope is the default) -->
<bean id="accountService" class="com.foo.DefaultAccountService" scope="singleton"/>
```

# The prototype scope

非单例的，原型的Bean指的就是每次请求Bean实例的时候，返回的都是新实例的Bean对象。也就是说，每次注入到另外的Bean或者通过调用getBean()来获得的Bean都将是全新的实例。 这是基于线程安全性的考虑，如果使用有状态的Bean对象用原型作用域，而无状态的Bean对象用单例作用域。  
下面的例子说明了Spring的原型作用域。DAO通常不会配置为原型对象，因为典型的DAO是不会有任何的状态的。

![多例](/img/2018-11-21/prototype.png)

下面的例子展示了XML中如何定义一个原型的Bean：

```
<bean id="accountService" class="com.foo.DefaultAccountService" scope="prototype"/>
```
与其他的作用域相比，Spring是不会完全管理原型Bean的生命周期的：Spring容器只会初始化，配置以及装载这些Bean，传递给Client。但是之后就不会再去管原型Bean之后的动作了。 也就是说，初始化生命周期回调方法在所有作用域的Bean是都会调用的，但是销毁生命周期回调方法在原型Bean是不会调用的。所以，客户端代码必须注意清理原型Bean以及释放原型Bean所持有的一些资源。 可以通过使用自定义的bean post-processor来让Spring释放掉原型Bean所持有的资源。  

在某些方面来说，Spring容器的角色就是取代了Java的new操作符，所有的生命周期的控制需要由客户端来处理。


---

