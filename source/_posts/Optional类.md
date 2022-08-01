---
title: Optional类
date: 2018-10-25 09:41:18
tags: [java]
---

# 介绍

```
Optional 类是一个可以为null的容器对象。如果值存在则isPresent()方法会返回true，调用get()方法会返回该对象。

Optional 是个容器：它可以保存类型T的值，或者仅仅保存null。 
Optional提供很多有用的方法，这样我们就不用显式进行空值检测。

Optional 类的引入很好的解决空指针异常。
```
<!--more-->

---

# 类方法描述

```
序号	方法 & 描述
1	static <T> Optional<T> empty()
返回空的 Optional 实例。

2	boolean equals(Object obj)
判断其他对象是否等于 Optional。

3	Optional<T> filter(Predicate<? super <T> predicate)
如果值存在，并且这个值匹配给定的 predicate，返回一个Optional用以描述这个值，否则返回一个空的Optional。

4	<U> Optional<U> flatMap(Function<? super T,Optional<U>> mapper)
如果值存在，返回基于Optional包含的映射方法的值，否则返回一个空的Optional

5	T get()
如果在这个Optional中包含这个值，返回值，否则抛出异常：NoSuchElementException

6	int hashCode()
返回存在值的哈希码，如果值不存在 返回 0。

7	void ifPresent(Consumer<? super T> consumer)
如果值存在则使用该值调用 consumer , 否则不做任何事情。

8	boolean isPresent()
如果值存在则方法会返回true，否则返回 false。

9	<U>Optional<U> map(Function<? super T,? extends U> mapper)
如果有值，则对其执行调用映射函数得到返回值。如果返回值不为 null，则创建包含映射返回值的Optional作为map方法返回值，否则返回空Optional。

10	static <T> Optional<T> of(T value)
返回一个指定非null值的Optional。

11	static <T> Optional<T> ofNullable(T value)
如果为非空，返回 Optional 描述的指定值，否则返回空的 Optional。

12	T orElse(T other)
如果存在该值，返回值， 否则返回 other。

13	T orElseGet(Supplier<? extends T> other)
如果存在该值，返回值， 否则触发 other，并返回 other 调用的结果。

14	<X extends Throwable> T orElseThrow(Supplier<? extends X> exceptionSupplier)
如果存在该值，返回包含的值，否则抛出由 Supplier 继承的异常

15	String toString()
返回一个Optional的非空字符串，用来调试
```

# 例子

```
package javaversion8;

import java.util.ArrayList;
import java.util.List;
import java.util.Optional;

/**
 * @author cxc
 * @date 2018/10/25 09:37
 * Java 8 Optional 类
 * Optional 类的引入很好的解决空指针异常。
 */
public class OptionalTest {

    public static void main(String[] args) {
        //返回空的 Optional 实例。
        Optional<Object> empty = Optional.empty();

        //判断其他对象是否等于Optional
        Integer A = 1;
        boolean equals = empty.equals(A);
        System.out.println("equals:" + equals);

        // Optional.ofNullable - 允许传递为 null 参数
        Integer value1 = null;
        Optional<Integer> a = Optional.ofNullable(value1);

        // Optional.of - 如果传递的参数是 null，抛出异常 NullPointerException
        Integer value2 = 1897001;
        Optional<Integer> b = Optional.of(value2);
        System.out.println("Optional.of:" + b.get());

        // Optional.isPresent - 判断值是否存在
        System.out.println("isPresent第一个参数值存在: " + a.isPresent());
        System.out.println("isPresent第二个参数值存在: " + b.isPresent());

        //void ifPresent(Consumer<? super T> consumer) 存在则进行消费 Lambda表达式
        b.ifPresent(System.out::println);

        //Optional.orElse - 如果值存在，返回它，否则返回默认值
        Integer value3 = a.orElse(new Integer(0));
        Integer value4 = b.orElse(new Integer(0));
        System.out.println("Optional.orElse:" + value3);
        System.out.println("Optional.orElse:" + value4);

        //T orElseGet(Supplier<? extends T> other)如果存在该值，返回值， 否则触发 other，并返回 other 调用的结果。
        Integer value8 = a.orElseGet(() -> 10086);
        System.out.println("orElseGet:" + value8);


        //Optional.get - 获取值，值需要存在 不存在会报错
        Integer value5 = b.get();
        System.out.println("Optional.get:" + value5);


        //Optional.of 传入null会报错NullPointerException
        Integer value6 = 6;
        Optional<Integer> c = Optional.of(value6);
        System.out.println("Optional.of :" + c.get());

        //map的使用,
        OptionalUser optionalUser = new OptionalUser("小明", "没有地址");
        String test = Optional.ofNullable(optionalUser).map(data -> data.getName()).map(data -> data.replace("小", "明")).orElse("default");
        System.out.println("Map:" + test);

        //过滤 filter
        List list = new ArrayList();
        list.add(1);
        list.add(2);
        list.add(3);
        Optional<List> list1 = Optional.of(list).filter(data -> data.size() > 2);
        list1.ifPresent(Data -> {
            Data.forEach(tet -> System.out.println("遍历内容" + tet));
        });


    }
}

```


# orElse() 和 orElseGet() 的不同之处
这个示例中，两个 Optional  对象都包含非空值，两个方法都会返回对应的非空值。不过，orElse() 方法仍然创建了 User 对象。与之相反，orElseGet() 方法不创建 User 对象。

在执行较密集的调用时，比如调用 Web 服务或数据查询，这个差异会对性能产生重大影响。