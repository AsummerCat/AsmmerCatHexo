---
title: vue.js组件基础
date: 2018-11-08 14:25:33
tags: [vue]
---

# 组件基础

```
这里有一个 Vue 组件的示例：

// 定义一个名为 button-counter 的新组件
Vue.component('button-counter', {
  data: function () {
    return {
      count: 0
    }
  },
  template: '<button v-on:click="count++">You clicked me {{ count }} times.</button>'
})
```

<!--more-->  

组件是可复用的 Vue 实例，且带有一个名字：在这个例子中是 <button-counter>。我们可以在一个通过 new Vue 创建的 Vue 根实例中，把这个组件作为自定义元素来使用：


```
<div id="components-demo">
  <button-counter></button-counter>
</div>
new Vue({ el: '#components-demo' })
```

---

* 需要注意的是:  
<font color="red">组件可以复用</font>

```
组件里面的data 必须是个functio(){}  
不然可能导致其他组件会用到相同的数据  
取而代之的是，一个组件的 data 选项必须是一个函数，因此每个实例可以维护一份被返回对象的独立的拷贝：

data: function () {
  return {
    count: 0
  }
}
```

---

# 通过 Prop 向子组件传递数据

大意就是 在组件中自定义一个参数  当一个值传入后会根据模板做出相应的操作  
当一个值传递给一个 prop 特性的时候，它就变成了那个组件实例的一个属性。

例子:

```
Vue.component('blog-post', {
  props: ['title'],
  template: '<h3>{{ title }}</h3>'
})

一个 prop 被注册之后，你就可以像这样把数据作为一个自定义特性传递进来：

<blog-post title="My journey with Vue"></blog-post>
<blog-post title="Blogging with Vue"></blog-post>
<blog-post title="Why Vue is so fun"></blog-post>


然而在一个典型的应用中，你可能在 data 里有一个博文的数组：

new Vue({
  el: '#blog-post-demo',
  data: {
    posts: [
      { id: 1, title: 'My journey with Vue' },
      { id: 2, title: 'Blogging with Vue' },
      { id: 3, title: 'Why Vue is so fun' }
    ]
  }
})
并想要为每篇博文渲染一个组件：

<blog-post
  v-for="post in posts"
  v-bind:key="post.id"
  v-bind:title="post.title"
></blog-post>
如上所示，你会发现我们可以使用 v-bind 来动态传递 prop。这在你一开始不清楚要渲染的具体内容，比如从一个 API 获取博文列表的时候，是非常有用的。
```

## 如果需要传入多个元素

两种方式
### 1.直接添加多个接收的参数 

```
Vue.component('blog-post', {
  props: ['title','max'],
  template: '<h3>{{ title }}<span v-html="max"></span></h3>'
})
```


### 2.接收一整个参数 在模板中进行解析

```
Vue.component('blog-post', {
  props: ['posts1'],
  template: '<h3>{{ posts1.title }}<span v-html="posts1.max"></span></h3>'
})
```

---

# 通过事件向父级组件发送消息

类似就是 比如说 全局放大文章内容  
例如我们可能会引入一个可访问性的功能来放大博文的字号，同时让页面的其它部分保持默认的字号。  

在其父组件中，我们可以通过添加一个 postFontSize 数据属性来支持这个功能：

```
new Vue({
  el: '#blog-posts-events-demo',
  data: {
    posts: [/* ... */],
    postFontSize: 1
  }
})
```

它可以在模板中用来控制所有博文的字号：

```
<div id="blog-posts-events-demo">
  <div :style="{ fontSize: postFontSize + 'em' }">
    <blog-post
      v-for="post in posts"
      v-bind:key="post.id"
      v-bind:post="post"
    ></blog-post>
  </div>
</div>
```

现在我们在每篇博文正文之前添加一个按钮来放大字号： 

```
Vue.component('blog-post', {
  props: ['post'],
  template: `
    <div class="blog-post">
      <h3>{{ post.title }}</h3>
      <button>
        Enlarge text
      </button>
      <div v-html="post.content"></div>
    </div>
  `
})
```
问题是这个按钮不会做任何事： 

```
<button>
  Enlarge text
</button>
```

当点击这个按钮时，我们需要告诉父级组件放大所有博文的文本。幸好 Vue 实例提供了一个自定义事件的系统来解决这个问题。我们可以调用内建的 `$emit `方法并传入事件的名字，来向父级组件触发一个事件：

```
<button v-on:click="$emit('enlarge-text')">
  Enlarge text
</button>
```

然后我们可以用 v-on 在博文组件上监听这个事件，就像监听一个原生 DOM 事件一样：

```
<blog-post
  ...
  v-on:enlarge-text="postFontSize += 0.1"></blog-post>
```

//重点  v-on:enlarge-text    
v-on:click="$emit('enlarge-text')

----

# 使用事件抛出一个值


有的时候用一个事件来抛出一个特定的值是非常有用的。例如我们可能想让 `<blog-post>` 组件决定它的文本要放大多少。这时可以使用 $emit 的第二个参数来提供这个值：

```
<button v-on:click="$emit('enlarge-text', 0.1)">
  Enlarge text
</button>
```
然后当在父级组件监听这个事件的时候，我们可以通过 <font color="red">$event</font> 访问到被抛出的这个值：

```
<blog-post
  ...
  v-on:enlarge-text="postFontSize += $event"></blog-post>
或者，如果这个事件处理函数是一个方法：

<blog-post
  ...
  v-on:enlarge-text="onEnlargeText"></blog-post>
那么这个值将会作为第一个参数传入这个方法：

methods: {
  onEnlargeText: function (enlargeAmount) {
    this.postFontSize += enlargeAmount
  }
}

```



