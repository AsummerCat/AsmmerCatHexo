---
title: JS时间格式转换
date: 2018-09-20 22:02:14
tags: 前端
​---

## JS时间格式转换:

```
function getMyDate(time){
    if(typeof(time)=="undefined"){
        return "";
    }
    var oDate = new Date(time),
        oYear = oDate.getFullYear(),
        oMonth = oDate.getMonth()+1,
        oDay = oDate.getDate(),
        oTime = oYear +'-'+ oMonth +'-'+oDay;//最后拼接时间
    return oTime;
};
```

