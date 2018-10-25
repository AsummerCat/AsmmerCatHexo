---
title: java8时间日期api
date: 2018-10-25 16:32:49
tags: java
---

# 介绍

```
在旧版的 Java 中，日期时间 API 存在诸多问题，其中有：

非线程安全 − java.util.Date 是非线程安全的，所有的日期类都是可变的，这是Java日期类最大的问题之一。

设计很差 − Java的日期/时间类的定义并不一致，在java.util和java.sql的包中都有日期类，此外用于格式化和解析的类在java.text包中定义。  
java.util.Date同时包含日期和时间，而java.sql.Date仅包含日期，将其纳入java.sql包并不合理。  
另外这两个类都有相同的名字，这本身就是一个非常糟糕的设计。

时区处理麻烦 − 日期类并不提供国际化，没有时区支持，因此Java引入了java.util.Calendar和java.util.TimeZone类，但他们同样存在上述所有的问题。

Java 8 在 java.time 包下提供了很多新的 API。以下为两个比较重要的 API：

Local(本地) − 简化了日期时间的处理，没有时区的问题。

Zoned(时区) − 通过制定的时区处理日期时间。

新的java.time包涵盖了所有处理日期，时间，日期/时间，时区，时刻（instants），过程（during）与时钟（clock）的操作。  

```
<!--more-->

# 例子:

日期/时间 : LocalDateTime  
日期:  LocalDate  
时间: LocalTime


```
import java.time.LocalDate;
import java.time.LocalDateTime;
import java.time.LocalTime;
import java.time.Month;
import java.time.format.DateTimeFormatter;

/**
 * @author cxc
 * @date 2018/10/25 16:31
 * java8 日期Api
 */
public class DateTimeUtils {

    public static void main(String[] args) {
        testLocalDateTime();
    }

    /**
     * 本地
     */
    private static void testLocalDateTime() {
        //核心
        LocalDateTime localDateTime = LocalDateTime.now();

        System.out.println("当前完整时间:" + localDateTime);

        LocalDate localDate = localDateTime.toLocalDate();
        System.out.println("当前年月:" + localDate);
        LocalTime localTime = localDateTime.toLocalTime();
        System.out.println("当前时间:" + localTime);
        LocalTime localTime1 = localDateTime.toLocalTime().withNano(0);
        System.out.println("当前时间(去除毫秒):" + localTime1);


        int year = localDateTime.getYear();
        System.out.println("年:" + year);
        int month = localDateTime.getMonth().getValue();
        System.out.println("月:" + month);
        int dayOfMonth = localDateTime.getDayOfMonth();
        System.out.println("日->在月中的位置:" + dayOfMonth);
        int dayOfWeek = localDateTime.getDayOfWeek().getValue();
        System.out.println("日->在星期中的位置:" + dayOfWeek);
        int dayOfYear = localDate.getDayOfYear();
        System.out.println("日->在年中的位置:" + dayOfYear);

        LocalDate date3 = LocalDate.of(2014, Month.of(11), 12);
        System.out.println("设置时间点: " + date3);

        // 22 小时 15 分钟
        LocalTime date4 = LocalTime.of(22, 15);
        System.out.println("date4: " + date4);

        // 解析字符串
        LocalTime date5 = LocalTime.parse("20:15:30");
        System.out.println("date5: " + date5);

        //转日期时间类型 比如    ..必须有日期+时间 不然会报错
        LocalDateTime parse = LocalDateTime.parse("2016-10-10 12:30:10", DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss"));
        System.out.println("转日期时间类型" + parse);

        //转日期类型 也是  必须只有日期
        LocalDate parse1 = LocalDate.parse("2018-10-12", DateTimeFormatter.ISO_DATE);
        System.out.println("转日期类型" + parse1);


    }

}

```
