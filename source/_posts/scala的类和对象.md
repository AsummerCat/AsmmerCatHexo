---
title: scala的类和对象
date: 2022-08-18 17:44:38
tags: [大数据,scala]
---
# scala的类和对象

# 基础语法
跟java类似,
但是不需要public修饰符,默认就是public
```
[修饰符] class 类名{
    类体
}
```
一个.scala源文件下可以有多个类

## 属性
<!--more-->
构造方法
```
@BeanProperty :符合javaBean规范
_ 表示引用值 (初始值);

[修饰符] class 类名{
定义属性
    @BeanProperty
    private var name:String= _
    @BeanProperty
    private var age: Int= _
}
```

## 构造方法
辅助构造器 必须用`this`这个关键字
```
class 类名(形参列表){ //主构造器
    //类体
    
   def this(形参列表){ //辅助构造器
       
   }
   def this(形参列表){ //辅助构造器可以有多个
       
   }
   
}
```
注意: 辅助构造器,不能直接构建对象,比如直接或者间接调用主构造方法


#### 在scala中使用这种推荐构造方法
```
//定义参数 并且有个默认的构造方法
class 类名(var name: String,var age: Int)


等价

class 类名{
定义属性
    @BeanProperty
    private var name:String= _
    @BeanProperty
    private var age: Int= _
}
```

#### 重载方法
```

override def xxx(): Unit ={
    //方法内容
}
```

## 抽象属性和抽象方法
```
(1) 定义抽象类 abstract class Person{} //通过abstract关键字标记抽象类

(2) 定义抽象属性 : val|var name: String //一个属性没有初始化,就是抽象属性

(3)定义抽象方法: def hello():String //只声明而没有实现的方法,就是抽象方法 
```


## 单例对象 语法
使用`object` 关键字声明
```
object Person{
    val coutry:String="China"
}
```

## 特质(Trait) 等于java的接口
#### 基本语法
```
trait 特质名{
    trait主体
    def xxx(): Unit
    def xxx1(): Unit    
}
```
#### 实操语法
```
没有父类: claas 类名 extends 特质1 with 特质2 with 特质3 ...

没有父类: claas 类名 extends 父类 with 特质2 with 特质3 ...
```
指定调用哪个父类方法
```
super[特质].xxx方法()
```

#### 对于接口的实现依赖注入
```
语法:   _: Dao =>

表示指向引用当前类UserDao\

trait UserDao{
    _: dao => 
    def xxx(): Unit
    def xxx1(): Unit    
}

```