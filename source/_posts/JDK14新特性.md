---
title: JDK14新特性
date: 2022-08-01 13:53:37
tags: [jdk新特性]
---
# JDK14新特性

## 空指针异常精准提示
通过 JVM 参数中添加-XX:+ShowCodeDetailsInExceptionMessages，可以在空指针异常中获取更为详细的调用信息，更快的定位和解决问题。
```
a.b.c.i = 99; // 假设这段代码会发生空指针
```
Java 14 之前：
<!--more-->
```
Exception in thread "main" java.lang.NullPointerException
    at NullPointerExample.main(NullPointerExample.java:5)
```
Java 14 之后：
```
 // 增加参数后提示的异常中很明确的告知了哪里为空导致
Exception in thread "main" java.lang.NullPointerException:
        Cannot read field 'c' because 'a.b' is null.
    at Prog.main(Prog.java:5)
```

## switch 的增强(转正)
Java12 引入的 switch（预览特性）在 Java14 变为正式版本，不需要增加参数来启用，直接在 JDK14 中就能使用。

Java12 为 switch 表达式引入了类似 lambda 语法条件匹配成功后的执行块，不需要多写 break ，Java13 提供了 yield 来在 block 中返回值。
```
String result = switch (day) {
            case "M", "W", "F" -> "MWF";
            case "T", "TH", "S" -> "TTS";
            default -> {
                if(day.isEmpty())
                    yield "Please insert a valid day.";
                else
                    yield "Looks like a Sunday.";
            }

        };
System.out.println(result);
```