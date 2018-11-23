---
title: Spring实例化容器
date: 2018-11-21 14:45:22
tags: Spring
---

# 实例化容器

实例化 Spring IoC 容器是直截了当的。提供给 ApplicationContext构造器的路径就是实际的资源字符串，使容器装入从各种外部资源的配置元数据，如本地文件系统， Java CLASSPATH，等等。

```
ApplicationContext context =
    new ClassPathXmlApplicationContext(new String[] {"services.xml", "daos.xml"});
```


<!--more--> 

# 使用容器
ApplicationContext 是能够保持 bean 定义以及相互依赖关系的高级工厂接口。使用方法 T getBean(String name, Class requiredType)就可以取得 bean 的实例。
ApplicationContext 中可以读取 bean 定义并访问它们，如下所示：

```
// create and configure beans
ApplicationContext context =
    new ClassPathXmlApplicationContext(new String[] {"services.xml", "daos.xml"});

// retrieve configured instance
PetStoreService service = context.getBean("petStore", PetStoreService.class);

// use configured instance
List<String> userList = service.getUsernameList();
```

# Bean 总览

Spring IoC 容易管理一个或者多个 bean。 bean 由应用到到容器的配置元数据创建,例如,在 XML 中定义 <bean/> 的形式。  
容器内部,这些 bean 定义表示为 BeanDefinition 对象,其中包含(其他信息)以下元数据:  
限定包类名称:典型的实际实现是定义 bean 的类。
bean 行为配置元素,定义了容器中的Bean应该如何行为(范围、生命周期回调,等等)。  
bean 需要引用其他 bean 来完成工作,这些引用也称为合作者或依赖关系。  
其他配置设置来设置新创建的对象,例如,连接使用 bean 的数量管理连接池,或者池的大小限制。  
---

# 命名bean

在基于 xml 的配置中,您可以使用 id 和(或)名称属性指定 bean 标识符。(id 属性允许您指定一个 id。  
通常这些名字使用字母数字(“myBean”、“fooService”,等等),但可以包含特殊字符。如果你想使用bean别名,您可以在name属性上定义它们,由逗号(,),分号(;),或白色空格进行分隔。作为一个历史因素的要注意,在 Spring 3.1 版本之前,id 属性被定义为 xsd:ID类型,它限制可能的字符。3.1,它被定义为一个 xsd:string 类型。注意,bean id 独特性仍由容器执行,虽然不再由 XML 解析器。


---

# 使用静态工厂方法实例化

当采用静态工厂方法创建 bean 时，除了需要指定 class 属性外，还需要通过 factory-method 属性来指定创建 bean 实例的工厂方法。Spring将调用此方法(其可选参数接下来介绍)返回实例对象，就此而言，跟通过普通构造器创建类实例没什么两样。  
下面的 bean 定义展示了如何通过工厂方法来创建bean实例。注意，此定义并未指定返回对象的类型，仅指定该类包含的工厂方法。在此例中，createInstance() 必须是一个 static 方法。

```
<bean id="clientService"
    class="examples.ClientService"
    factory-method="createInstance"/>
```

```
public class ClientService {
    private static ClientService clientService = new ClientService();
    private ClientService() {}

    public static ClientService createInstance() {
        return clientService;
    }
}
```

给工厂方法指定参数以及为bean实例设置属性的详细内容请查阅依赖和配置详解。  

使用实例工厂方法实例化  
 
与通过 静态工厂方法 实例化类似，通过调用工厂实例的非静态方法进行实例化。 使用这种方式时，class属性置为空，而factory-bean属性必须指定为当前(或其祖先)容器中包含工厂方法的bean的名称，而该工厂bean的工厂方法本身必须通过factory-method属性来设定。

```
<!-- 工厂bean，包含createInstance()方法 -->
<bean id="serviceLocator" class="examples.DefaultServiceLocator">
    <!-- 其他需要注入的依赖项 -->
</bean>

<!-- 通过工厂bean创建的ben -->
<bean id="clientService"
    factory-bean="serviceLocator"
    factory-method="createClientServiceInstance"/>
```

```
public class DefaultServiceLocator {

    private static ClientService clientService = new ClientServiceImpl();
    private DefaultServiceLocator() {}

    public ClientService createClientServiceInstance() {
        return clientService;
    }
}
```


# 依赖注入

依赖注入是一个让对象只通过构造参数，工厂方法的参数或者配置的属性来定义他们的依赖的过程。这些依赖也是对象所需要协同工作的对象。容器会在创建Bean的时候注入这些依赖。整个过程完全反转了由Bean自己控制实例化或者引用依赖，所以这个过程也称之为控制反转。

当使用了依赖注入的准则以后，会更易于管理和解耦对象之间的依赖，使得代码更加的简单。对象不再关注依赖，也不需要知道依赖类的位置。这样的话，开发者的类更加易于测试，尤其是当开发者的依赖是接口或者抽象类的情况，开发者可以轻易在单元测试中mock对象。

<font color="red">依赖注入主要使用两种方式，一种是基于构造函数的注入，另一种的基于Setter方法的依赖注入。</font>

## 构造函数注入

基于构造函数的依赖注入是由IoC容器来调用类的构造函数，构造函数的参数代表这个Bean所依赖的对象。跟调用带参数的静态工厂方法基本一样。下面的例子展示了一个类通过构造函数来实现依赖注入的。需要注意的是，这个类没有任何特殊的地方，只是一个简单的，不依赖于任何容器特殊接口，基类或者注解的普通类。

```
public class SimpleMovieLister {
    private MovieFinder movieFinder;
    public SimpleMovieLister(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

}
```

### 构造函数的参数解析
构造函数的参数解析是通过参数的类型来匹配的。如果在Bean的构造函数参数不存在歧义，那么构造器参数的顺序也就是就是这些参数实例化以及装载的顺序。参考如下代码：

```
package x.y;

public class Foo {

    public Foo(Bar bar, Baz baz) {
        // ...
    }

}
```
假设Bar和Baz在继承层次上不相关，也没有什么歧义的话，下面的配置完全可以工作正常，开发者不需要再去<constructor-arg>元素中指定构造函数参数的索引或类型信息。

```
<beans>
    <bean id="foo" class="x.y.Foo">
        <constructor-arg ref="bar"/>
        <constructor-arg ref="baz"/>
    </bean>

    <bean id="bar" class="x.y.Bar"/>

    <bean id="baz" class="x.y.Baz"/>
</beans>
```
当引用另一个Bean的时候，如果类型确定的话，匹配会工作正常（如上面的例子）.当使用简单的类型的时候，比如说<value>true</value>，Spring IoC容器是无法判断值的类型的，所以是无法匹配的。考虑代码如下：

```
package examples;

public class ExampleBean {
    private int years;

    public ExampleBean(int years, String ultimateAnswer) {
        this.years = years;
        this.ultimateAnswer = ultimateAnswer;
    }

}
```

在上面代码这种情况下，容器可以通过使用构造函数参数的type属性来实现简单类型的匹配。比如：

```
<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg type="int" value="7500000"/>
    <constructor-arg type="java.lang.String" value="42"/>
</bean>
```

或者使用index属性来指定构造参数的位置，比如：

```
<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg index="0" value="7500000"/>
    <constructor-arg index="1" value="42"/>
</bean>
```

这个索引也同时是为了解决构造函数中有多个相同类型的参数无法精确匹配的问题。需要注意的是，索引是基于0开始的。 开发者也可以通过参数的名称来去除二义性。

```
<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg name="years" value="7500000"/>
    <constructor-arg name="ultimateAnswer" value="42"/>
</bean>
```

需要注意的是,做这项工作的代码必须启用了调试标记编译,这样Spring才可以从构造函数查找参数名称。开发者也可以使用@ConstructorProperties注解来显式声明构造函数的名称，比如如下代码：

```
package examples;

public class ExampleBean {

    @ConstructorProperties({"years", "ultimateAnswer"})
    public ExampleBean(int years, String ultimateAnswer) {
        this.years = years;
        this.ultimateAnswer = ultimateAnswer;
    }

}
```

---

## Set方法注入
基于Setter函数的依赖注入则是容器会调用Bean的无参构造函数，或者无参数的工厂方法，然后再来调用Setter方法来实现的依赖注入。  
下面的例子展示了使用Setter方法进行的依赖注入，下面的类对象只是简单的POJO对象，不依赖于任何Spring的特殊的接口，基类或者注解。

```
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }
}
```

ApplicationContext所管理Bean对于基于构造函数的依赖注入，或者基于Setter方式的依赖注入都是支持的。同时也支持使用Setter方式在通过构造函数注入依赖之后再次注入依赖。开发者在BeanDefinition中可以使用PropertyEditor实例来自由选择注入的方式。然而，大多数的开发者并不直接使用这些类，而是跟喜欢XML形式的bean定义，或者基于注解的组件（比如使用@Component，@Controller等）或者在配置了@Configuration的类上面使用@Bean的方法。

>基于构造函数还是基于Setter方法？   
>因为开发者可以混用两者，所以通常比较好的方式是通过构造函数注入必要的依赖通过Setter方式来注入一些可选的依赖。其中，在Setter方法上面的@Required注解可用来构造必要的依赖。 Spring队伍推荐基于构造函数的注入，因为这种方式会促使开发者将组件开发成不可变对象而且确保了注入的依赖不为null。而且，基于构造函数的注入的组件被客户端调用的时候也是完全构造好的。当然，从另一方面来说，过多的构造函数参数也是非常差的代码方式，这种方式说明类貌似有了太多的功能，最好重构将不同职能分离。 基于Setter的注入只是用于可选的依赖，但是也最好配置一些合理的默认值。否则，需要对代码的依赖进行非NULL的检查了。基于Setter方法的注入有一个便利之处在于这种方式的注入是可以进行重配置和重新注入的。 依赖注入的两种风格适合大多数的情况，但是有时使用第三方的库的时候，开发者可能并没有源码，而第三方的代码也没有setter方法，那么就只能使用基于构造函数的依赖注入了。



使用Setter方式进行依赖注入。代码如下：

```
<bean id="exampleBean" class="examples.ExampleBean">
    <!-- setter injection using the nested ref element -->
    <property name="beanOne">
        <ref bean="anotherExampleBean"/>
    </property>

    <!-- setter injection using the neater ref attribute -->
    <property name="beanTwo" ref="yetAnotherBean"/>
    <property name="integerProperty" value="1"/>
</bean>

<bean id="anotherExampleBean" class="examples.AnotherBean"/>
<bean id="yetAnotherBean" class="examples.YetAnotherBean"/>
```

```
public class ExampleBean {

    private AnotherBean beanOne;
    private YetAnotherBean beanTwo;
    private int i;

    public void setBeanOne(AnotherBean beanOne) {
        this.beanOne = beanOne;
    }

    public void setBeanTwo(YetAnotherBean beanTwo) {
        this.beanTwo = beanTwo;
    }

    public void setIntegerProperty(int i) {
        this.i = i;
    }

}
```

下面的例子，是通过静态的工厂方法来返回Bean实例的。

```
<bean id="exampleBean" class="examples.ExampleBean" factory-method="createInstance">
    <constructor-arg ref="anotherExampleBean"/>
    <constructor-arg ref="yetAnotherBean"/>
    <constructor-arg value="1"/>
</bean>

<bean id="anotherExampleBean" class="examples.AnotherBean"/>
<bean id="yetAnotherBean" class="examples.YetAnotherBean"/>
```

```
public class ExampleBean {
    private ExampleBean(...) {
        ...
    }

    // a static factory method; the arguments to this method can be
    // considered the dependencies of the bean that is returned,
    // regardless of how those arguments are actually used.
    public static ExampleBean createInstance (
        AnotherBean anotherBean, YetAnotherBean yetAnotherBean, int i) {

        ExampleBean eb = new ExampleBean (...);
        // some other operations...
        return eb;
    }

}
```

---

# Bean懒加载

默认情况下，ApplicationContext会在实例化的过程中创建和配置所有的单例Bean。总的来说，这个预初始化是很不错的。因为这样能及时发现环境上的一些配置错误，而不是系统运行了很久之后才发现。如果这个行为不是迫切需要的，开发者可以通过将Bean标记为延迟加载就能阻止这个预初始化。延迟初始化的Bean会通知IoC不要让Bean预初始化而是在被引用的时候才会实例化。 在XML中，可以通过<bean/>元素的lazy-init属性来控制这个行为。如下：

```
<bean id="lazy" class="com.foo.ExpensiveToCreateBean" lazy-init="true"/>
<bean name="not.lazy" class="com.foo.AnotherBean"/>
```

当将Bean配置为上面的XML的时候，ApplicationContext之中的lazyBean是不会随着ApplicationContext的启动而进入到预初始化状态的，而那些非延迟加载的Bean是处于预初始化的状态的。
然而，如果一个延迟加载的类是作为一个单例非延迟加载的Bean的依赖而存在的话，ApplicationContext仍然会在ApplicationContext启动的时候加载，因为作为单例Bean的依赖，会随着单例Bean的实例化而实例化。

---

# 自动装配

Spring容器可以根据Bean之间的依赖关系自动装配。开发者可以令Spring通过ApplicationContext来来自动解析这些关联。自动的装载有很多的优点：

* 自动装载能够明显的减少指定的属性或者是构造参数。
* 自动装载可以扩展开发者的对象。比如说，如果开发者需要加一个依赖，依赖就能够不需要开发者特别关心更改配置就能够自动满足。这样，自动装载在开发过程中是极度高效的，不用明确的选择装载的依赖会使系统更加的稳定。




