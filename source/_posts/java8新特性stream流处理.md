---
title: java8新特性stream流处理
date: 2019-11-21 15:31:16
tags: java
---

# java8新特性stream流处理

## 新特性ForEach

```java
peoples.forEach(people -> System.out.println(people.getName()));
```

## 过滤年龄大于18的人员

```java
List<Person> collect = peoples.stream().filter(person -> person.getAge() > 18).collect(toList());
```

<!--more-->

## 过滤属性

比如获取`list<user> `下面的用户名称 单独做个list

```java
	List<String> collect = peoples.stream().map(Person::getName).collect(toList());
```

## 分组统计

```java
   //分组统计  根据名称分组 并且 根据house聚合计算
Map<String, IntSummaryStatistics> map=  peoples.stream().collect(groupingBy(Person::getName,summarizingInt(Person::getHouse)));
```

```java
  //分组统计  根据名称分组 分成相同名称的集合
		Map<String, List<Person>> map=  peoples.stream().collect(groupingBy(Person::getName));
```

## 去重 (需要重写eques 和hashcode 的方法)

```
 // 去重复，也比较常用，但是需要重写eques 和hashcode 的方法
        List<Person> collect2 = streamSupplier.get().distinct().collect(toList());
```



## 限制条数，做分页可以使用

```
List<Person> collect3 = streamSupplier.get().limit(4).collect(toList());
```



##  统计数量

```
long count = stream.filter(p -> p.getAge() > 20).count();
```

## 规约

聚合计算

```java
Optional<Integer> reduce = streamSupplier.get().filter(s -> s.getAge() < 30).map(Person::getAge).reduce(Integer::sum);
        Integer reduce1 = streamSupplier.get().filter(s -> s.getAge() < 40).map(Person::getAge).reduce(0,(a,b)->a+b);
        Integer reduce2 = streamSupplier.get().filter(s -> s.getAge() < 40).map(Person::getAge).reduce(0,(a,b)->a+b);
        Integer reduce3 = streamSupplier.get().filter(s -> s.getAge() < 40).map(Person::getAge).reduce(2,(a,b)->a+b);
       
返回值:        
66
66
68
```

## 获取流对象

```
获取流对象
List<Person> list = new ArrayList<Person>(); 
Stream<Person> stream = list.stream();
对于数组来说,通过Arrays类提供的静态函数stream()获取数组的流对象
String[] names = {"chaimm","peter","john"};
Stream<String> stream = Arrays.stream(names);
直接将几个普通的数值变成流对象
Stream<String> stream = Stream.of("chaimm","peter","john");

```



## 获取最大最小值

##  max和min

```java
 Artist theMaxAgeArtist = allArtists.stream()
                                       .max(Comparator.comparing(artist -> artist.getAge()))
                                       .get();
```



# 坑点

注意：流操作一个流只能进行一次处理操作，像下面这个的做法就会出现问题。

```
   List<Person> peoples = new ArrayList<>(Arrays.asList(
                new Person("思聪","杭州",19,50),
                new Person("马云","北京",50,100),
                new Person("思聪","北京",19,20),
                new Person("温州大婶","西安",1,120),
                new Person("温州大婶","杭州",1,100),
                new Person("我",null,18,0),
                new Person("温州大婶","新西兰",1,200)
        ));

        //+++++ 坑点+++++
        Stream<Person> stream = peoples.stream();
        List<String> collect = stream.map(Person::getName).collect(toList()); // 第一次使用 stream
        List<Person> collect1 = stream.filter(person -> person.getAge() > 18).collect(toList());// 第二次使用 stream

        System.out.println();
```

异常：
`java.lang.IllegalStateException: stream has already been operated upon or closed`
解决方式：

## 有解决方案：

` Supplier<Stream<Person>> streamSupplier = peoples::stream;`

```java
   @Test
    public void testMyErr() throws Exception {
        List<Person> peoples = new ArrayList<>(Arrays.asList(
                new Person("思聪","杭州",19,50),
                new Person("马云","北京",50,100),
                new Person("思聪","北京",19,20),
                new Person("温州大婶","西安",1,120),
                new Person("温州大婶","杭州",1,100),
                new Person("我",null,18,0),
                new Person("温州大婶","新西兰",1,200)
        ));

        //+++++ 坑点+++++
//        Stream<Person> stream = peoples.stream();
//        List<String> collect = stream.map(Person::getName).collect(toList()); // 第一次使用 stream
//        List<Person> collect1 = stream.filter(person -> person.getAge() > 18).collect(toList());// 第二次使用 stream
//
        // 类似于一个stream的池子，用的时候取，每次都是个新的对象
        Supplier<Stream<Person>> streamSupplier = peoples::stream;
        List<String> collect3 = streamSupplier.get().map(Person::getName).collect(toList()); // 第一次使用 stream
        List<Person> collect4 =streamSupplier.get().filter(person -> person.getAge() > 18).collect(toList());// 第二次使用 stream
        System.out.println();
    }
```

