---
title: qlexpress规则引擎之QL脚本说明
date: 2022-12-13 16:04:39
tags: [规则引擎,qlexpress]
---
和java语法相比，要避免的一些ql写法错误
不支持try{}catch{}
注释目前只支持 /** **/，不支持单行注释 //
不支持java8的lambda表达式
不支持for循环集合操作for (Item item : list)
弱类型语言，请不要定义类型声明,更不要用Template（Map<String, List>之类的）
array的声明不一样
min,max,round,print,println,like,in 都是系统默认函数的关键字，请不要作为变量名
//java语法：使用泛型来提醒开发者检查类型
<!--more-->
```
keys = new ArrayList<String>();
deviceName2Value = new HashMap<String, String>(7);
String[] deviceNames = {"ng", "si", "umid", "ut", "mac", "imsi", "imei"};
int[] mins = {5, 30};
```
//ql写法：
```
keys = new ArrayList();
deviceName2Value = new HashMap();
deviceNames = ["ng", "si", "umid", "ut", "mac", "imsi", "imei"];
mins = [5, 30];
```
//java语法：对象类型声明
```
FocFulfillDecisionReqDTO reqDTO = param.getReqDTO();
//ql写法：
reqDTO = param.getReqDTO();
```
//java语法：数组遍历
```
for(Item item : list) {
}
//ql写法：
for(i = 0; i < list.size(); i++){
item = list.get(i);
}
```
//java语法：map遍历
```
for(String key : map.keySet()) {
System.out.println(map.get(key));
}
//ql写法：
keySet = map.keySet();
objArr = keySet.toArray();
for (i = 0; i < objArr.length; i++) {
key = objArr[i];
System.out.println(map.get(key));
}
```