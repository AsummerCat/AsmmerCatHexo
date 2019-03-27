---
title: orcle切割字符串函数方法
date: 2019-03-24 20:36:46
tags: [数据库,orcle]
---

# orcle切割字符串函数方法 

约等于列转行

## 需要注意的是 这种切割方式只能切割单条数据 

## 函数

```
 --参数 : 需要切割字符串 , 正则 (这边为切割',') ,行数

SELECT REGEXP_SUBSTR ('需要切割字符串', '[^,]+', 1,rownum) from dual   
connect by rownum<=LENGTH ('需要切割字符串') - LENGTH (regexp_replace('需要切割字符串', ',', ''))+1;
```

