---
title: ElasticSearch笔记实现锁并发控制(二十七)
date: 2020-08-31 21:56:57
tags: [ElasticSearch笔记]
---

# 注意:
需要注意的是:es的锁基本上都是两步操作的
```
类似于请求一次加锁操作->再请求一次增删改操作
```
不是严格意义上的锁  
因为加锁这个操作相当于在另外一个type里标注了该doc ,如果你直接操作该doc是可以成功,所以你需要在每一步操作的时候去进行判断锁是否相同

## 基于document锁实现并发
三种锁实现:
1. 悲观锁  (5.x之前的版本) 之后被移除了
2. 共享锁
3. 排他锁


<!--more-->

# 新建一个索引专门用来lock
注意:7.0之后如果还想用type需要加入`include_type_name=true`
```
PUT /fs/lock/_mapping?include_type_name=true
{
  "properties": {
    
  }
}
```

# 基于悲观锁实现   (5.x之前的版本) 之后被移除了
固定语法 ` /index/lock`: 表示,index下的lock type 专门用来进行上锁操作的
### 悲观锁加锁脚本
首先在`/es/config/scripts/目录里放入groovy脚本`  
脚本内容:

```
脚本名称:judge-lock.groovy 

内容:
if (ctx._source.precess_id != process_id) {assert false}; ctx.op='noop';
```
es 6.0之后默认的写法
```
POST _scripts/document-lock
{
  "script": {
    "lang": "painless",
    "source": "if ( ctx._source.process_id != params.process_id ) { Debug.explain('already locked by other thread'); }  ctx.op = 'noop';"
  }
}
```


### 悲观锁加锁操作:
```
POST /fs/lock/1/_update
{
  "upsert": {"process_id": 123},
  "script": {
    "lang": "groovy",
    "file": "judge-lock",
    "params":{
      "process_id":123
    }
  }
}
```
1. <font color="red">process_id</font>,是你应用进程的唯一id,比如可以自己用UUID生成一个唯一ID   
比如说 :java中给操作es的线程赋予一个UUID,表示该线程执行 `process_id+thread_id` 用来唯一标识
2. <font color="red">assert false</font> 不是当前进程加锁的话,则抛出异常
3. <font color="red">ctx.op='noop'</font>  不作任何修改

### 原理 
如果该document之前没有被锁,就是说id=1的doc不存在,name执行insert操作,创建一个lock锁,process_id被设置为123,script不执行.  

如果document被说了,就是说id=1的doc存在,name执行update操作,此时会比对`process_id`,   
如果相同,说明是该进程之前锁的doc,则不报错,不执行任何操作,返回<font color="red">success</font>  
如果process_id对不上,说明doc被其他doc给锁了,此时报错  
有报错的话,如果有些doc被锁了,那么需要重试  
知道所有锁定都成功,执行自己的操作....


### 解锁
```
PUT /fs/lock/_bulk
{"delete":{"id":1}}

或者
delete /fs/lock/1
```


# 共享锁 (读锁)
### 共享锁加锁脚本
判断是否是排他锁,是的话报错,不是的话 lock_count+1
```
POST _scripts/gx-lock
{
  "script": {
    "lang": "painless",
    "source":"if ( ctx._source.lock_type == 'exclusive' ) { Debug.explain('already locked by other thread'); } ctx._source.lock_count++;"
 }
}
```

这种语法不好用,还是用groovy比较好
```
if ( ctx._source.lock_type == 'exclusive' ) { assert false }; ctx._source.lock_count++;
```

### 加锁语法

```
POST /fs/lock/1/_update
{
    "upsert":{
        "lock_type":"shared",
        "lock_count":1
    },
    "script":"gx-lock"
}

```

### 共享锁解锁脚本
```
POST _scripts/gx-unlock
{
  "script": {
    "lang": "painless",
    "source":"if (--ctx._source.lock_count == 0) { ctx.op = 'delete'} "
}
}

```
### 共享锁解锁语法
```
POST /fs/lock/1/_update
{
  "script": {
    "id": "gx-unlock"
  }
}
```


# 排他锁 (写锁)
排他锁->用的不是`upsert`语法, 用的是`create`语法,要求lock必须不能存在,直接自己是第一个上锁的人,上的排他锁

## 排他锁写法
如果存在其他锁 直接报错
```
PUT /fs/lock/1/_create
{
  "lock_type":"exclusive"
}

```
## 删除锁 写法
直接delete就可以了
```
delete /fs/lock/1
```