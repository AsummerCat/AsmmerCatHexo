---
title: scala基础语法
date: 2022-08-18 17:42:45
tags: [大数据,scala]
---
# scala基础语法

### scala安装
跟java类似 需要jvm平台,和配置环境变量

### 语法结构
```
// object: 关键字,声明一个单例对象(伴生对象)
object HelloWorld{
    
    /
     * def 方法名称(参数名称: 参数类型): 返回值类型={
     *   方法体
     * }
     */
    def main(args: Array[String]) :Unit ={
        println("hello world");
    }
}
```
<!--more-->
ps : `Unit` 等价于java的`void`返回值

### 定义对象
注意:伴生对象必须跟对象名一致,而且是在同一个文件里
```
//既声明属性 又是构造方法

class Student(var name: String ,var age: Int){
    def xxx方法(): Unit = {
        //方法体
    }
}

//使用伴生对象 来使用静态方法 或者属性 ->类.方法
object Student{
    //静态属性
    val school: String ="类静态属性"
    
    //静态方法
    def xxx方法(): Unit = {
        //方法体
    }
    
    比如main方法就在这里运行
}

```

# 变量

#### 变量定义var 与常量 val
注意 类型可以自动推导 不用指定
比如: `var i=10`
```
//变量
var 变量名[:变量属性] =初始值 
var i:Int =10

//常量
val 常量名[:常量属性] =初始值
val i:Int =10

```

## 字符串模板
通过`$`获取变量值  s开头
```
println(s"${age}岁的${name}在学习")
```
## 格式化字符串模板
通过`$`获取变量值  f开头,可自定义格式 比如保留小数
```
保留2位整数,2位小数
var age = 2.3456l

println(f"${age}%2.2f岁的${name}在学习")
```

## 字符串模板 原始输出
变化部分仅修改变量
通过`$`获取变量值  raw开头
```
println("${age}岁的${name}在学习")

```

### 多行显示

```
使用 """来表示

比如:


s"""
  | select * 
  | from
  | student
  | where
  | name =${name}
  |and
  | age >${age}
  |""".stripMargin
  
```

## IO操作
#### 读取文件
```

//1. 从文件中读取数据, 并遍历按行打印
Source.fromFile("文件路径/1.txt").foreach(print)
```

#### 写入文件
```
//2.将数据写入文件
val writer= new PrintWriter(new File("文件路径/1.txt"))
writer.write("hello world")
writer.close()
```


## if判断
无返回值
```
var res: Unit = if (age >=18){
    print("成年")
}else{
    print("未成年")
}

```
有返回值
```
var res: String = if (age >=18){
   "成年"
}else{
    "未成年"
}
```

## 循环控制
#### 包含循环
```
包含1 2 3
for(i <- 1 to 3){
    print(i+" ")
}

(1) i表示循环的遍历, <- 规定to
(2) i将会从1-3循环,前后闭合

```
#### 不包含循环 Range 或者 until
```
不包含10
 Range(1,10,1):
参数: 
起始位置
结束位置
步
长
for(i <- Range(1,10,1)){
    print(i+" ")
}


(1) i将会从1-10循环,不包含10


或者用这种语法
for(i <- 1 until 10){
    print(i+" ")
}

```

#### 集合遍历
```
for(i <- Array(12,13,14)){
    print(i)
}

for(i <- List(12,13,14)){
    print(i)
}

for(i <- Set(12,13,14)){
    print(i)
}
```

#### 循环守卫 类似java的continue
跳过部分条件不符合的
直接在if里写
```
for(i <- Set(12,13,14) if i! = 13){
    print(i)
}
最终输出 12 14

等价
for(i <- Set(12,13,14)){
    if(i!=13){
        print(i)
    }
}
```

#### 循环步长 语法 by
输出 1 3 4 7 9 步长为2
```
for(i <- 1 to 10 by 2){
    print(i)
}
```

#### 遍历反转 reverse
从10遍历到1
```
for(i <- 1 to 10 reverse){
    print(i)
}
```
#### 嵌套for循环
```
for(i<- 1 to 4; j <- 1 to 5){
    print("i="+i,"j="+j)
}


等价
for(i<- 1 to 3){
    for(j <- 1 to 3){
        print("i="+i,"j="+j)
    }
}
```
#### 循环返回值 关键字yield 很少用
将循环过程中的值返回到一个list中
```
val res = for(i <- 1 to 10) yield i
println(res)
```

# 异常捕获 try
```
语法:
try{
    
}catch{
    case e=> Exception => 执行内容
}

```