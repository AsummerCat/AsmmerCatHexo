---
title: go循环语句
date: 2022-10-19 13:57:28
tags: [go]
---
# go循环语句

## for循环
```
		sum := 0
		for i := 0; i <= 10; i++ {
			sum += i
		}
		fmt.Println(sum)
```
<!--more-->
## 无限循环
```
		//无限循环
	for {
		sum++ // 无限循环下去
	}
```

## For-each range 循环
```
   strings := []string{"google", "runoob"}
   for index, s := range strings {
      fmt.Println(index, s)
   }
```