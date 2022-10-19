---
title: Go的time包
date: 2022-10-19 14:03:52
tags: [go]
---
# Go的time包

## 时间类型
```
time.Time 类型表示时间
我们可以通过 time.Now()函数来获取当前的时间对象
```
<!--more-->

## 案例
```
package main

import (
	"fmt"
	"time"
)

func main() {
	timeDemo()
}

func timeDemo() {
	now := time.Now() //获取当前时间
	fmt.Println(now)

	year := now.Year()     //年
	month := now.Month()   //月
	day := now.Day()       //日
	hour := now.Hour()     //小时
	minute := now.Minute() //分钟
	second := now.Second() //秒

	fmt.Println(year, month, day, hour, minute, second)
}

```


## 时间戳
```
package main

import (
	"fmt"
	"time"
)

func main() {
	timestampDemo()
}

func timestampDemo() {
	now := time.Now() //获取当前时间
	fmt.Println(now)

	unix := now.Unix()     //时间戳
	nano := now.UnixNano() //纳秒时间戳

	fmt.Println(unix)
	fmt.Println(nano)

	//将时间戳转换为时间
	nowDate := time.Unix(unix, 0)
	fmt.Println(nowDate)
}


```

## 时间间隔
```
time定义常量

time.Minute 1分钟
time.Hour 1小时

time.Duration 表示1纳秒
time.Second 表示1秒
```

# 时间相关操作
## add
```
func (t Time ) Add (d Duration) Time
```
## sub求差值
```
sub := addTime.Sub(now)
```

## Before 在xxx之前
```
func (t Time)Before(u Time) bool
如果t表示的时间在u之前,返回真,否则返回假
```
## after 在xxx之后
```
func (t Time)After(u Time) bool
如果t表示的时间在u之后,返回真,否则返回假
```

## 定时器
```
使用time.Tick(时间间隔)来设置定时器

tick := time.Tick(time.Second) //定义一个1秒间隔的定时器
	for i:=range tick{
		fmt.Println(i)//每秒都会执行的任务
	}
}
```
## 时间格式化
```
	now := time.Now()
	//格式化的模板为Go的出生日期 2006年1月2号15点04分

	//24小时制
	fmt.Println(now.Format("2006-01-02 15:04:05.000 Mon Jan"))
	//12小时制
	fmt.Println(now.Format("2006-01-02 3:04:05.000 PM Mon Jan"))

	fmt.Println(now.Format("2006/01/02"))
	fmt.Println(now.Format("2006/01/02 :15:04:05"))
	fmt.Println(now.Format("15:04 \"2006/01/02"))
```

# 完整案例
```
package main

import (
	"fmt"
	"time"
)

/*
*
基于时间的操作
*/
func main() {
	//追加时间
	addDemo()
	//求差值
	subDemo()
	//在xxx时间之前
	BeforeDemo()
	//在xxx时间之后
	AfterDemo()
	//时间格式化
	TimeFormat()
	//定时器
	TimeTaskDemo()
}

/*
时间加时间间隔 add
*/
func addDemo() time.Time {
	now := time.Now() //获取当前时间
	//加3小时
	addtime := now.Add(time.Hour * 3)
	fmt.Println(addtime)
	return addtime
}

/*
求两个时间之间的差值
*/
func subDemo() {
	now := time.Now() //获取当前时间
	addTime := addDemo()
	sub := addTime.Sub(now)
	fmt.Println(sub)
}

/*
判断时间是否在xxx之前
*/
func BeforeDemo() {
	now := time.Now() //获取当前时间
	addTime := addDemo()
	before := now.Before(addTime)
	fmt.Println(before)
}

/*
判断时间是否在xxx之前
*/
func AfterDemo() {
	now := time.Now() //获取当前时间
	addTime := addDemo()
	after := now.After(addTime)
	fmt.Println(after)
}

/*
定时器 time.Tick(时间间隔)
*/
func TimeTaskDemo() {
	tick := time.Tick(time.Second) //定义一个1秒间隔的定时器
	for i := range tick {
		fmt.Println(i) //每秒都会执行的任务
	}
}

/*
时间格式化 Format
*/
func TimeFormat() {
	now := time.Now()
	//格式化的模板为Go的出生日期 2006年1月2号15点04分

	//24小时制
	fmt.Println(now.Format("2006-01-02 15:04:05.000 Mon Jan"))
	//12小时制
	fmt.Println(now.Format("2006-01-02 3:04:05.000 PM Mon Jan"))

	fmt.Println(now.Format("2006/01/02"))
	fmt.Println(now.Format("2006/01/02 :15:04:05"))
	fmt.Println(now.Format("15:04 \"2006/01/02"))
}

```
