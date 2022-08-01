---
title: mysql查询当前时间，一天内，一周，一个月内的sql语句
date: 2018-10-31 16:27:45
tags:  [数据库]
---

# 语法 函数从日期减去指定的时间间隔。

* 定义和用法

* `DATE_SUB() `函数从日期减去指定的时间间隔。

* `DATE_SUB(date,INTERVAL expr type)`

<!--more-->

# 语法2 返回周期P1和P2之间的月数。

*  `PERIOD_DIFF(P1,P2)` 返回周期P1和P2之间的月数。 P1和P2格式为YYMM或YYYYMM。
*  注意周期参数 P1 和 P2 都不是日期值。

# 参数类型

```
date 参数是合法的日期表达式。expr 参数是您希望添加的时间间隔。

type 参数可以是下列值：

MICROSECOND
SECOND
MINUTE
HOUR
DAY
WEEK
MONTH
QUARTER
YEAR
SECOND_MICROSECOND
MINUTE_MICROSECOND
MINUTE_SECOND
HOUR_MICROSECOND
HOUR_SECOND
HOUR_MINUTE
DAY_MICROSECOND
DAY_SECOND
DAY_MINUTE
DAY_HOUR
YEAR_MONTH
```

# 例子

## 减去2天
```
SELECT OrderId,DATE_SUB(OrderDate,INTERVAL 2 DAY) AS OrderPayDate
FROM Orders
```

## 查询一天

```
select * from 表名 where to_days(时间字段名) = to_days(now());
```

## 昨天

```
SELECT * FROM 表名 WHERE TO_DAYS( NOW( ) ) - TO_DAYS( 时间字段名) <= 1
```

## 七天内

```
SELECT * FROM 表名 where DATE_SUB(now(), INTERVAL 7 DAY) <= date(时间字段名)
```

## 查询时间节点一周内数据

```
select * from Tabel名 where 时间字段名 between current_date()-7 and sysdate()
```

## 近乎30天

```
SELECT * FROM 表名 where DATE_SUB(CURDATE(), INTERVAL 30 DAY) <= date(时间字段名)
```

## 本月

```
SELECT * FROM 表名 WHERE DATE_FORMAT( 时间字段名, '%Y%m' ) = DATE_FORMAT( CURDATE( ) , '%Y%m' )
```

## 上一月

```
SELECT * FROM 表名 WHERE PERIOD_DIFF( date_format( now( ) , '%Y%m' ) , date_format( 时间字段名, '%Y%m' ) ) =1
```