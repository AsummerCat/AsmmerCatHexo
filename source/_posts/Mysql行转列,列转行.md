---
layout: mysql
title: 行转列
date: 2018-09-17 20:35:55
tags: [mysql,数据库]
categories: 数据库
---


## 1.行转列语法
>
SELECT GROUP_CONCAT(字段,':',字段) as image_url FROM t_visitor_info 




