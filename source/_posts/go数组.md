---
title: go数组
date: 2022-10-19 13:59:02
tags: [go]
---
# go数组

## 声明数组
```
var variable_name [SIZE] variable_type

例如:
var balance [10] float32
```

<!--more-->
## 初始化数组
```
var balance = [5]float32{1000.0, 2.0, 3.4, 7.0, 50.0}

不确定长度
var balance = [...]float32{1000.0, 2.0, 3.4, 7.0, 50.0}

//  将索引为 1 和 3 的元素初始化
balance := [5]float32{1:2.0,3:7.0}
```

## 访问数组元素
```
var salary float32 = balance[9]
```

## 二维数组
```
sites := [2][2]string{}

    // 向二维数组添加元素
    sites[0][0] = "Google"
    sites[0][1] = "Runoob"
    sites[1][0] = "Taobao"
    sites[1][1] = "Weibo"
    
    
    a := [3][4]int{  
 {0, 1, 2, 3} ,   /*  第一行索引为 0 */
 {4, 5, 6, 7} ,   /*  第二行索引为 1 */
 {8, 9, 10, 11},   /* 第三行索引为 2 */
}
```

## 例子
```
package main

import "fmt"

func main() {
	var n [10]int /* n 是一个长度为 10 的数组 */
	var i, j int

	/* 为数组 n 初始化元素 */
	for i = 0; i < 10; i++ {
		n[i] = i + 100 /* 设置元素为 i + 100 */
	}

	/* 输出每个数组元素的值 */
	for j = 0; j < 10; j++ {
		fmt.Printf("Element[%d] = %d\n", j, n[j])
	}

	/*二维数组*/
	// 创建二维数组
	sites := [2][2]string{}

	// 向二维数组添加元素
	sites[0][0] = "Google"
	sites[0][1] = "Runoob"
	sites[1][0] = "Taobao"
	sites[1][1] = "Weibo"

	// 显示结果
	fmt.Println(sites)
}

```

## 传递数组
```
package main

import "fmt"

func main() {
   /* 数组长度为 5 */
   var  balance = [5]int {1000, 2, 3, 17, 50}
   var avg float32

   /* 数组作为参数传递给函数 */
   avg = getAverage( balance, 5 ) ;

   /* 输出返回的平均值 */
   fmt.Printf( "平均值为: %f ", avg );
}
func getAverage(arr [5]int, size int) float32 {
   var i,sum int
   var avg float32  

   for i = 0; i < size;i++ {
      sum += arr[i]
   }

   avg = float32(sum) / float32(size)

   return avg;
}
```
