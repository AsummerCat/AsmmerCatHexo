---
title: go的Map数组
date: 2022-10-19 13:59:54
tags: [go]
---
# go的Map数组

## 创建集合
```
 var countryCapitalMap map[string]string /*创建集合 */
    countryCapitalMap = make(map[string]string)
```
## 查看元素在集合中是否存在
```
capital, ok := countryCapitalMap [ "American" ]
ok 表示:是否存在
capital 表示: 值

```
<!--more-->
## 删除元素
delete(countryCapitalMap, "France")
```
delete() 
```
## 案例
```
package main

import "fmt"

func main() {
	var countryCapitalMap map[string]string /*创建集合 */
	countryCapitalMap = make(map[string]string)
	/* map插入key - value对,各个国家对应的首都 */
	countryCapitalMap["France"] = "巴黎"
	countryCapitalMap["Italy"] = "罗马"
	countryCapitalMap["Japan"] = "东京"
	countryCapitalMap["India "] = "新德里"

	/*使用键输出地图值 */
	for country := range countryCapitalMap {
		fmt.Println(country, "首都是", countryCapitalMap[country])
	}
	/*查看元素在集合中是否存在 */
	capital, ok := countryCapitalMap["American"] /*如果确定是真实的,则存在,否则不存在 */
	/*fmt.Println(capital) */
	/*fmt.Println(ok) */
	if ok {
		fmt.Println("American 的首都是", capital)
	} else {
		fmt.Println("American 的首都不存在")
	}
	delete(countryCapitalMap, "France")
}

```