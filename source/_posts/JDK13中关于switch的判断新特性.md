---
title: JDK13中关于switch的判断新特性
date: 2022-02-09 22:28:37
tags: [java,jdk新特性]
---

# JDK13中关于switch的判断新特性
## 新增yield关键字 可以进行if判断
<!--more-->

## 代码
并且可以作为返回值返回  
yield 可以作为判断输出
```

static void howMany(int k) {
    System.out.println(
        switch (k) {
            case  1 -> "one"
            case  2 -> "two"
            default -> {
                if(k=3){
                yield "three";
                }
                if(k=4){
                yield "fire";
                }
             }
            }
    );
}

```
并且 
```
case 1,2 可以表示 1和2的结果 输出为一个
```
