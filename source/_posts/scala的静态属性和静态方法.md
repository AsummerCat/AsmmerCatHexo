---
title: scala的静态属性和静态方法
date: 2022-08-18 17:44:08
tags: [大数据,scala]
---
# scala的静态属性和静态方法
```
object Accompany {
  def main(args: Array[String]): Unit = {

    println(ScalaPerson.sex)  //true 在底层等价于ScalaPerson$.MODULE$.sex()
    ScalaPerson.sayHi() //在底层等价于ScalaPerson$.MODULE$.sex()
  }
}

//说明
//1.当在同一个文件中，有class ScalaPerson 和 object  ScalaPerson
//2.class ScalaPerson 称为伴生类，将非静态的内容写到该类中来
//3.object ScalaPerson 称为伴生对象，将静态的内容写入到该对象（类）
//4.class ScalaPerson编译后底层生成ScalaPerson类ScalaPerson.class
//5.object ScalaPerson编译后底层生成ScalaPerson$类ScalaPerson$.class
//6.对于伴生对象的内容，我们可以直接通过ScalaPerson.属性 或者 方法

//伴生类
class ScalaPerson {
  var name : String = _
}

//伴生对象
object ScalaPerson {
  var sex: Boolean = true
  def sayHi(): Unit = {
    println("object ScalaPerson syaHi")
  }
}

```