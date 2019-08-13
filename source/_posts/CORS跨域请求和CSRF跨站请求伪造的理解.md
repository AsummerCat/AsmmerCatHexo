---
title: CORS跨域请求和CSRF跨站请求伪造的理解
date: 2019-08-13 23:21:56
tags: [java,前端,跨域]
---

# CORS跨域请求和CSRF跨站请求伪造的理解

## CORS 跨域

所谓CORS，是指通过XMLHttpRequest的ajax方式访问其他域名下资源，而不是在A域名的页面上点击打开一个B域名的页面。引入不同域上的js脚本文件也是没问题的。出于安全考虑，跨域请求不能访问document.cookie对象。

当一个请求url的协议、域名、端口三者之间任意一与当前页面地址不同即为跨域。

```
比如 [http://www.aaa.com](http://www.aaa.com/) 和下列URL相比，都不属于同源。

[https://www.aaa.com](https://www.aaa.com/)

[http://www.aaa.com:8080](http://www.aaa.com:8080/)

[http://aaa.com](http://aaa.com/)
```

<!--more-->

但下面这种属于同源:

http://username:password@[www.aaa.com](http://www.aaa.com/)

例如最常见的，在一个域名下的网页上，调用另一个域名中的资源。

实际效果演示，读者可以点击[这里](http://jsbin.com/fusaweqe/1/edit?html,js,output)，然后打开开发者工具，查看报错信息。

CORS技术允许跨域访问多种资源，比如javascript，字体文件等，这种技术对XMLHttpRequest做了升级，使之可以进行跨域访问。但不是所有的浏览器都支持CORS技术。Firefox和Chrome等浏览器支持的比较好，稍微新一点的版本都支持。IE比较搓，IE10才真正支持这个机制，IE10以下需要用XDomainRequest这个对象，这是IE特有的。

当然不是说浏览器支持了就立刻可以跨域访问了，CORS技术中最重要的关键点是响应头里的Access-Control-Allow-Origin这个Header。 此Header是W3C标准定义的用来检查是可否接受跨域请求的一个标识



```java
返回结果中加上如下控制字段：

Access-Control-Allow-Origin: 允许跨域访问的域，可以是一个域的列表，也可以是通配符"*"。这里要注意Origin规则只对域名有效，并不会对子目录有效。即http://foo.example/subdir/ 是无效的。但是不同子域名需要分开设置，这里的规则可以参照同源策略

Access-Control-Allow-Credentials: 是否允许请求带有验证信息，这部分将会在下面详细解释

Access-Control-Expose-Headers: 允许脚本访问的返回头，请求成功后，脚本可以在XMLHttpRequest中访问这些头的信息(貌似webkit没有实现这个)

Access-Control-Max-Age: 缓存此次请求的秒数。在这个时间范围内，所有同类型的请求都将不再发送预检请求而是直接使用此次返回的头作为判断依据，非常有用，大幅优化请求次数

Access-Control-Allow-Methods: 允许使用的请求方法，以逗号隔开

Access-Control-Allow-Headers: 允许自定义的头部，以逗号隔开，大小写不敏感

然后浏览器通过返回结果的这些控制字段来决定是将结果开放给客户端脚本读取还是屏蔽掉。如果服务器没有配置cors，返回结果没有控制字段，浏览器会屏蔽脚本对返回信息的读取。
```



## CSRF跨站请求伪造

```java
在他们的钓鱼站点，攻击者可以通过创建一个AJAX按钮或者表单来针对你的网站创建一个请求：

<form action="https://my.site.com/me/something-destructive" method="POST">
  <button type="submit">Click here for free money!</button>
</form>
这是很危险的，因为攻击者可以使用其他http方法例如 delete 来获取结果。 这在用户的session中有很多关于你的网站的详细信息时是相当危险的。 如果一个不懂技术的用户遇到了，他们就有可能会输入信用卡号或者个人安全信息。
```



