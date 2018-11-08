---
title: vue.js计算属性+方法+监听器
date: 2018-11-08 09:36:47
tags: vue.js
---
# 计算属性 computed

```javascript
var vm = new Vue({
  el: '#example',
  data: {
    message: 'Hello'
  },
  computed: {
    // 计算属性的 getter
    reversedMessage: function () {
      // `this` 指向 vm 实例
      return this.message.split('').reverse().join('')
    }
  }
})
```

​     <font color="red">这里跟methods 有点不一样 </font>

## 计算属性默认有get方法



```javascript
计算属性默认只有 getter ，不过在需要时你也可以提供一个 setter ：

// ...
computed: {
  fullName: {
    // getter
    get: function () {
      return this.firstName + ' ' + this.lastName
    },
    // setter
    set: function (newValue) {
      var names = newValue.split(' ')
      this.firstName = names[0]
      this.lastName = names[names.length - 1]
    }
  }
}
// ...
```



---

 方法中的this 是指向实例的

---

  

# 方法 methods

```javascript
// 在组件中
methods: {
  reversedMessage: function () {
    return this.message.split('').reverse().join('')
  }
}
```

<font color="red">tips: </font>  

<font color="red"> 这里需要注意的是 我们完全可以把一个函数构建成一个方法而不是计算属性 两种结果都一样 但是使用计算属性的时候是基于它们的依赖进行缓存的  如果没有改变 '参数'的值 那么返回的都是之前的结果 (缓存)    

而方法是每次都重新调用一次函数执行</font>  



```javascript
这也同样意味着下面的计算属性将不再更新，因为 Date.now() 不是响应式依赖：

computed: {
  now: function () {
    return Date.now()
  }
}
```

# 监听器

 这里可以用来监听 input输入情况 来发送ajax之类的



例子:

```javascript
<div id="watch-example">
  <p>
    Ask a yes/no question:
    <input v-model="question">
  </p>
  <p>{{ answer }}</p>
</div>

<script>
var watchExampleVM = new Vue({
  el: '#watch-example',
  data: {
    question: '',
    answer: 'I cannot give you an answer until you ask a question!'
  },
  watch: {
    // 如果 `question` 发生改变，这个函数就会运行
    question: function (newQuestion, oldQuestion) {
      this.answer = 'Waiting for you to stop typing...'
      this.debouncedGetAnswer()
    }
  },
  created: function () {
    // `_.debounce` 是一个通过 Lodash 限制操作频率的函数。
    // 在这个例子中，我们希望限制访问 yesno.wtf/api 的频率
    // AJAX 请求直到用户输入完毕才会发出。想要了解更多关于
    // `_.debounce` 函数 (及其近亲 `_.throttle`) 的知识，
    // 请参考：https://lodash.com/docs#debounce
    this.debouncedGetAnswer = _.debounce(this.getAnswer, 500)
  },
  methods: {
    getAnswer: function () {
      if (this.question.indexOf('?') === -1) {
        this.answer = 'Questions usually contain a question mark. ;-)'
        return
      }
      this.answer = 'Thinking...'
      var vm = this
      axios.get('https://yesno.wtf/api')
        .then(function (response) {
          vm.answer = _.capitalize(response.data.answer)
        })
        .catch(function (error) {
          vm.answer = 'Error! Could not reach the API. ' + error
        })
    }
  }
})
</script>

```

