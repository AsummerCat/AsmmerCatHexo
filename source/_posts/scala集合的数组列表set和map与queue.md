---
title: scala集合的数组列表set和map与queue
date: 2022-08-18 17:46:14
tags: [大数据,scala]
---
# scala集合
1) 分为三大类
   `序列Seq`,`集Set`,`映射Map`,所有集合都扩展自Iterable特质;

2) 对于几乎所有集合类,Scala都同时提供了可变和不可变的版本,分别位于一下两个包
```
不可变集合: scala.collection.immutable 
可变集合:   scala.collection.mutable
```
不可变集合: 该集合不可修改,每次修改都会返回一个新对象

可变集合: 可直接在原对象上进行修改,而不会返回新对象;

<!--more-->

## 不可变集合集成图
![image](/img/2022-08-15/1.png)

## 可变集合集成图
![image](/img/2022-08-15/2.png)

# 数组
#### 不可变数组 new Array
```
//1.创建不可变数组
val numArr =new Array[Int](10)

//或者
var numArr1=Array(12,13,14,15)

println(numArr.length)


```
###### 不可变元素 ->添加元素 方法  :+(元素)
```
var numArr1=Array(12,13,14,15)

//添加元素
var newArr = numArr.:+(73)
//或者
var newArr1 =19 +: 29 +: newArr :+26 :+73

//遍历拼接 使用-拼接
newArr.mkString("-")
```

#### 可变数组 new ArrayBuffer
默认有16的长度;
```
//1.创建可变数组
var arr1= new ArrayBuffer()
var arr1: ArrayBuffer[Int]= new ArrayBuffer[Int]()
var arr1=  ArrayBuffer(23,57,92)

```
###### 可变元素 ->添加元素 方法 +=元素
```
var arr1=  ArrayBuffer(23,57,92)

//末尾追加
arr1 +=13
//开头追加
 13 +=: arr1

//末尾追加元素值
arr1.append(2)
//开头追加元素值
arr1.prepend(2)

//任意位置之后追加元素 比如 在元素1的位置之后
arr1.insert(1,13,14)

//末尾追加另外一个数组 2表示在元素位置2之后追加
arr1.insertAll(2,newArr1)

//开头追加另外一个数组
arr1.prependAllAll(newArr1)

```
###### 可变元素->删除元素 remove
```
arr1.remove(索引位置)
arr1.remove(索引位置,删除索引位置之后个数)

//直接删除指定元素 ,删除14这个值的元素
arr1 -= 14

```

#### 不可变数组与可变数组转换
```
var arr1 =  ArrayBuffer(23,57,92)
//转换不可变数组
var arr2: Array[Int] = arr1.toArray

//转换为可变数组
var buffer: mutable.Buffer[Int] = arr2.toBuffer

```

#### 多维数组
```
//创建二维数组
var array : Array[Array[Int]] = Array.ofDim[Int](2,3)

//访问元素 例:第一行第二个元素
array(0)(2)

//赋值
array(0)(2) =25


```

# 列表list

#### 不可变List
```
//1. 创建一个不可变List
var list1 = List(23,65,87)
var list1 = 32 :: Nil
var list1 = 32:: 17 :: 28 :: Nil

//2. 遍历元素
list1.foreach(println)

//3.添加元素
//前面添加一个元素
var list2 = 10.+=: list1
//后面添加一个元素
var list3 = list1.:+ 10


//4;添加一个元素到开头 
//使用场景是Nil.::(13) 给空集合初始化元素
var list4 = list2.::(51)

//5.合并列表 ::: 三个冒号
var list9 = list6 ::: list7
或:
var list9 = list6 ++ list7


```

#### 可变ListBuffer
```
//1.创建可变列表
var list1 = new ListBuffer[Int]()
var list1 = new ListBuffer(12,13,14)

//2.添加元素
//尾部追加
list1.append(15)
//头部追加
list1.prepend(15)
//指定位置添加
list.install(1,42)

//3.合并list
var list3 = list1 ++ list2

//4. 修改元素
list2(3) = 30
list2.update(0,89)

//5.删除元素 remove
list.remove(2)

//6.删除指定元素值
list2 -= 25


```

# Set集合

#### 不可变set
```
//1.创建set
var set1 = Set(12,13,14)

//2.添加元素 +
var set2 =set1+ 20

//3.合并set ++
var set3 =set2 ++ set1

//4.删除元素
var set4 =set3 - 13

```

#### 可变set mutable.Set
```
//1.创建set
var set1: mutable.Set[Int]= mutable.Set(12,13,14)

//2.添加元素 add
set1.add(12)

//3.删除元素 remove
set1.remove(12)

//4.合并可变Set set1的内容追加到set3
set3 += set1


```

# Map
#### 不可变Map Map
```
//1.创建Map 
val map1 :Map[String,String] =Map()
//创建Map 并且给予初始值
val map1 :Map[String,String] =Map("key" ->"value","key1"->"value1")

//2.遍历元素
map1.foreach(println)

//3. 遍历Map中的所有key 或者value
for(key <- map1.keys){
    println(s"$key ->${map1.get(key)}")
}


//4. 访问某一个key的value
map1.get("a").get
map1.getOrElse("a",0)



```

#### 可变Map mutable.Map
```
//1.创建一个可变Map
val map1 :mutable.Map[String,Int] =mutable.Map()

//2. 遍历元素
map1.foreach(println)

for(key <- map1.keys){
    println(s"$key ->${map1.get(key)}")
}

//3.查询元素
map1.get("a").get
map1.getOrElse("a",0)

//4.添加元素
map1.put("c",5)
map1.put("d",9)

//5.删除元素
map1.remove("c")
map1.remove("d")

//6.map合并 将map2的元素追加到map1中
map1 ++=map2

```


# 元组 Tuple
注: 元组最大只能有22个元素
```
//1.创建元组

var tuple =(13,14,15,"hello","测试")

 //2. 访问数据 获取对应下标的内容 _1 开头
 println(tuple._1)
 println(tuple._2)
 println(tuple._3)
 //或者
 //获取索引位置上的值 从0开始 
 tuple.productElement(1)
 
 //2.遍历元组
 for (elem <- tuple.productIterator)
    println(elem)
```



# 队列queue
```
val que = new mutable.Queue[String]()

//入队
que.enqueue("a","b","c","d")

//出队
que.dequeue()
```
