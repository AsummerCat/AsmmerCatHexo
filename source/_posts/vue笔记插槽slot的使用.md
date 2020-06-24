---
title: vue笔记插槽slot的使用
date: 2020-06-24 22:32:21
tags: [vue]
---

# vue笔记插槽slot的使用
让组件更具有扩展性
利用外部 来扩展组件内部
让使用者来决定组件内部具体展示什么内容

## 使用方式
### 创建方式一
插槽:`<slot></slot>`
```
在组件中定义一个插槽位置 预留位

<template>
<slot></slot>
</template>

```
<!--more-->
### 创建方式二 含有默认内容的slot
如果插槽没被使用,显示默认插槽的内容
```
<template>
<slot><span>默认插槽内容</span></slot>
</template>

```

### 调用
如果只有一个插槽,不管多少内容都是一个插槽内的内容
```

# 组件
<cp>
# 插槽内容
<span>我是插槽内容</span>
</cp>

```


## 具名插槽 
具体名称的插槽

```
顾名思义 就是给slot添加name 用来标识
这样的话 如果有多个slot 可以选择填写其中的一个,其他的还是默认

<template>
<slot name="test1"><span>默认插槽内容1</span></slot>
<slot name="test2"><span>默认插槽内容2</span></slot>
<slot name="test3"><span>默认插槽内容3</span></slot>
</template>

```
### 使用:
如果说你写的是`<slot>内容`是不带name那么只会替换没有name的插槽
```
在你要写的标签里加入 slot="插槽的name" 就会去替换

比如:
# 组件
<cp>
# 插槽内容
<span slot="test1">我是自定义插槽内容1</span>
<span slot="test2">我是自定义插槽内容2</span>
</cp>

```

## 获取组件中的slot内容

### 组件内容
shuju是组件中定义的 :data传递
```
<template id="cpn">
  <slot :data="shuju">
    <ul>
      <li v-for="item in shuju">{{item}}</li>
    <ul>
  </slot>

<template>
```
### 使用
slot-scope="slot" 主要是为了获取组件传出的内容
```
<template slot-scope="slot">
  <span v-for="item in slot.data">{{item}}</span>
</template>
```