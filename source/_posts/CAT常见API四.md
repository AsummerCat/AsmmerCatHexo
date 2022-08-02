---
title: CAT常见API四
date: 2022-08-02 14:56:15
tags: [链路跟踪,cat]

---
# CAT常见API

## Transaction

### 基础用法
```
//开启一个事务, 类型为url, 名称为借口路径
        Transaction t = Cat.newTransaction(CatConstants.TYPE_URL, request.getRequestURL().toString());
        try {
          
            //执行目标方法
            
            //成功执行Transaction
            t.setStatus(Transaction.SUCCESS);
        } catch (Exception ex) {
        //失败设置异常
            t.setStatus(ex);
            Cat.logError(ex);
            throw ex;
        } finally {
        //关闭当前事务
            t.complete();
        }
```
<!--more-->
#### 扩展API
* addData 添加额外的数据显示
* setStatus 设置状态,成功可以设置SUCCESSS,失败可以设置异常
* setDurationInMillis 设置执行耗时(毫秒)
* setTimestamp 设置执行时间
* complete 结束Transaction

```
//开启一个事务, 类型为url, 名称为借口路径
        Transaction t = Cat.newTransaction(CatConstants.TYPE_URL, request.getRequestURL().toString());
        try {
         
            //修改执行耗时为1秒
         t.setDurationInMillis(1000L);
         
            //设置开始时间
        t.setTimestamp(System.currentTimeMillis);
        
            //添加额外的数据
        t.addData("context");
        
            //执行目标方法
            
            //成功执行Transaction
            t.setStatus(Transaction.SUCCESS);
        } catch (Exception ex) {
        //失败设置异常
            t.setStatus(ex);
            Cat.logError(ex);
            throw ex;
        } finally {
        //关闭当前事务
            t.complete();
        }
```

## Event
#### Cat.logEvent
记录一个事件
```
//参数1:type类型 参数2:名称 参数3:状态 参数4:打印信息字符串
Cat.logEvent("URL.server","serverIp",Event.SUCCESS,"ip=#{serverIp}");
```
#### Cat.logError
记录一个带有错误堆栈信息的Error
```
try{
    int i =1/0;
}catch(Exception e){
   Cat.logError(e);
}
```
或者
```
这样控制台会返回一个带前缀的异常
   Cat.logError("这是一个异常",e);
```

## Metric
记录一个业务指标,业务指标最低统计颗粒度为1分钟
```
//默认加1
Cat.logMerticForCount("metric.key");
//手动设置累计值为3
Cat.logMerticForCount("metric.key",3);

# Duration 累计值为5
Cat.logMetricForDuration("metric.key",5)
```
在`Duration`的情况下,用平均值来取代累计值
