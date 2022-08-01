---
title: List去重复和数组去重
date: 2018-09-20 22:02:14
tags: [java,工具类]
---
## 方式一

```
//如何去除字段b重复的对象呢?
        System.out.println(list.size());
        HashMap<String, String> map = new HashMap<String, String>();
        for(int i=0;i<list.size();i++){
        //根据map的key做判断
            if(map.get(list.get(i))!=null){
                list.remove(i);
            }else{
                map.put(list.get(i), "OK");
            }
        }
        System.out.println(list.size());
```
<!--more-->


## 方式二

```
private String[] StringjustNO1(String[] frist) {
    List list = Arrays.asList(frist);
    Set set = new HashSet(list);
    String[] rid = (String[]) set.toArray(new String[0]);
    return rid;
}
```

