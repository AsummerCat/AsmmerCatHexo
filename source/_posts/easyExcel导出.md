---
title: easyExcel导出
date: 2022-08-01 13:51:39
tags: [java]
---

# easyExcel导出


导出xlsx:
```
EasyExcel.write(response.getOutputStream(), clazz).sheet("sheet1").doWrite(data);
```
