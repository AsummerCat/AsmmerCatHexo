---
title: java后台接收json的方式
date: 2019-11-19 11:41:52
tags: [springmvc,java]
---

# java后台接收json的方式

## 以RequestParam接收

前端传来的是json数据不多时：[id:id],可以直接用@RequestParam来获取值

```java
@RequestMapping(value = "/update")
@ResponseBody
public String updateAttr(@RequestParam ("id") int id) {
    int res=id;
    return "success";
}

```

## 以实体类方式接收

前端传来的是一个json对象时：{【id，name】},可以用实体类直接进行自动绑定

```java
@RequestMapping(value = "/add")
    @ResponseBody
    public String addObj(@RequestBody Accomodation accomodation) {
       Accomodation a=accomodation;
        return "success";
    }
```

<!--more-->

## 以Map接收

前端传来的是一个json对象时：{【id，name】},可以用Map来获取

```java
@RequestMapping(value = "/update")
@ResponseBody
public String updateAttr(@RequestBody Map<String, String> map) {
    if(map.containsKey("id"){
        Integer id = Integer.parseInt(map.get("id"));
    }
    if(map.containsKey("name"){
        String objname = map.get("name").toString();
    }
    // 操作 ...
    return "success";
}

```

## 以List接收

当前端传来这样一个json数组：[{id,name},{id,name},{id,name},...]时，用List<E>接收

```java
@RequestMapping(value = "/update")
@ResponseBody
public String updateAttr(@RequestBody List<Accomodation> list) {
    for(Accomodation accomodation:list){
        System.out.println(accomodation.toString());
    }
    return "success";
}
```

