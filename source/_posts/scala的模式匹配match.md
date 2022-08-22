---
title: scala的模式匹配match
date: 2022-08-22 11:59:11
tags: [大数据,scala]
---
# scala的模式匹配match
类似java的switch

## 基础语法 match
```
var a : Int =10
var b : Int =20
//匹配值
var operator :char ='d'

var result = operator match{
    case '+' => a+b
    case '-' => a-b
    case '*' => a*b
    case '/' => a/b
    case _ => "默认值"    
}

println(result)
```
<!--more-->
## 模式匹配
表达匹配某个范围的数据,就需要在模式匹配中增加条件守卫,就是可以加上if判断

```
var operator :Int =10

//这里意思是 如果i >0 则返回i ,i表示传入的值
var result = operator match{
    case i if i >=0  => i
    case i if i <0  => -i
    case _ => "默认值"    
}
```
