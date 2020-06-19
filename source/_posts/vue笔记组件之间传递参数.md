---
title: vue笔记组件之间传递参数
date: 2020-06-19 17:26:08
tags: [vue]
---

# vue笔记组件之间传递参数

## 第一种 父传子 给组件传递参数 props
```
<组件 :组件中的参数="页面vue的date值"> </组件>

比如:


<template>
<mytest :heihei="msg"></mytest>
</template>

export default{
    name : 'app',
    date(){
        return{
            msg:'hello word'
        }
    }
    components:{
        'mytest': mytest
    }
}


在mytest.vue中用props接收参数
export default{
    name : 'mytest',
    //接收传递的参数
    props: ['heihei','heihei1']
}

这里就将msg 传递给mytest中的heihei属性了

```
通过子组件的props部分,指明可以用来接收其他外部的参数.父组件可以写在子组件的标签中 用来传递参数
<!--more-->

## props的两种写法
### 第一种
```
最简便的获取参数方式
props:[参数列表]
比如:
props:['test1','test2']

```
### 第二种 
```
设置具体传入的参数的类型
props:[键值对]
比如:
props:{
    myName:{
        type:String,
        required:true,
        default:'xx'
    }
    
}
```
## 参数传递 子传父
### 方式1
```
改用方法传递
父页面创建Function
<template>
<mytest :myBtn="fcn"></mytest>
</template>

export default{
    name : '父页面',
    methods:{
        fcn:function(m){
            this.msg=m;
        }
    }
}
这样的写法 大意就是 子组件调用该函数->父组件具体实现函数 
需要注意的是 父组件定义的this还是父组件的
从数据上看是子传父


子页面接收Function

<template>
>button @click="myBtn('hello word')"> 子页面的内容</button>
</template>

props:{
    'myBtn':{
     type:Function 
    }
    
}

```

### 方式二 常用 子传父
this.$emit('属性名',"属性值");传递给父vue
```
子页面: 
<template>
>button @click="xxxclick"> 子页面的内容</button>
</template>

methods:{
    xxxclick(){
        this.$emit('myName',给父页面的内容);
    }
}

父页面接收:

<mytest @myName="父页面的字段=$event"></mytest>


@myName为子页面发射的字段= 赋值给父页面的字段=$event

    
}


```