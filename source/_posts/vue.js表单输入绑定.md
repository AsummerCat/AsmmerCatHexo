---
title: vue.js表单输入绑定
date: 2018-11-08 13:21:35
tags: vue.js
---
# v-model=" " 双向数据绑定

你可以用 v-model 指令在表单` <input>、<textarea> 及 <select>` 元素上创建双向数据绑定。它会根据控件类型自动选取正确的方法来更新元素。尽管有些神奇，但 v-model 本质上不过是语法糖。它负责监听用户的输入事件以更新数据，并对一些极端场景进行一些特殊处理。

<font color="red">首先都需要在 new vue({}) 定义参数</font>

<!--more-->

## 文本

```
<input v-model="message" placeholder="edit me"/>
<p>Message is: {{ message }}</p>

```

## 多行文本

```
<span>Multiline message is:</span>
<p style="white-space: pre-line;">{{ message }}</p>
<br>
<textarea v-model="message" placeholder="add multiple lines"></textarea>
```

## 复选框

```
<input type="checkbox" id="checkbox" v-model="checked">
<label for="checkbox">{{ checked }}</label>
```
## 复选框单个也可以这么写 

```
<input
  type="checkbox" v-model="toggle" true-value="yes" false-value="no"
>
```

## 多个复选框绑定到同一个数组

```
<div id='example-3'>
  <input type="checkbox" id="jack" value="Jack" v-model="checkedNames">
  <label for="jack">Jack</label>
  <input type="checkbox" id="john" value="John" v-model="checkedNames">
  <label for="john">John</label>
  <input type="checkbox" id="mike" value="Mike" v-model="checkedNames">
  <label for="mike">Mike</label>
  <br>
  <span>Checked names: {{ checkedNames }}</span>
</div>

new Vue({
  el: '#example-3',
  data: {
    checkedNames: []
  }
})
```

## 单选按钮

```
<div id="example-4">
  <input type="radio" id="one" value="One" v-model="picked">
  <label for="one">One</label>
  <br>
  <input type="radio" id="two" value="Two" v-model="picked">
  <label for="two">Two</label>
  <br>
  <span>Picked: {{ picked }}</span>
</div>

new Vue({
  el: '#example-4',
  data: {
    picked: ''
  }
})

也可以这么写
<input type="radio" v-model="pick" v-bind:value="a">

这样的话就是选中的话 就会绑定 参数 a;

```

## 选择框
单选:

```
<div id="example-5">
  <select v-model="selected">
    <option disabled value="">请选择</option>
    <option>A</option>
    <option>B</option>
    <option>C</option>
  </select>
  <span>Selected: {{ selected }}</span>
</div>

new Vue({
  el: '...',
  data: {
    selected: ''
  }
})
```

多选:

```
<div id="example-6">
  <select v-model="selected" multiple style="width: 50px;">
    <option>A</option>
    <option>B</option>
    <option>C</option>
  </select>
  <br>
  <span>Selected: {{ selected }}</span>
</div>


new Vue({
  el: '#example-6',
  data: {
    selected: []
  }
})
```

---

遍历select 时候 绑定value

```
<select v-model="selected">
  <option v-for="option in options" v-bind:value="option.value">
    {{ option.text }}
  </option>
</select>
<span>Selected: {{ selected }}</span>


new Vue({
  el: '...',
  data: {
    selected: 'A',
    options: [
      { text: 'One', value: 'A' },
      { text: 'Two', value: 'B' },
      { text: 'Three', value: 'C' }
    ]
  }
})


```



# 修饰符


## .lazy
在默认情况下，v-model 在每次 input 事件触发后将输入框的值与数据进行同步 (除了上述输入法组合文字时)。你可以添加 lazy 修饰符，从而转变为使用 change 事件进行同步：

```
<!-- 在“change”时而非“input”时更新 -->
<input v-model.lazy="msg" >
```

## .number
如果想自动将用户的输入值转为数值类型，可以给 v-model 添加 number 修饰符:  
```
<input v-model.number="age" type="number">
```  
这通常很有用，因为即使在 type="number" 时，HTML 输入元素的值也总会返回字符串。如果这个值无法被 parseFloat() 解析，则会返回原始的值。

## .trim

如果要自动过滤用户输入的首尾空白字符，可以给 v-model 添加 trim 修饰符：

```
<input v-model.trim="msg">
```


