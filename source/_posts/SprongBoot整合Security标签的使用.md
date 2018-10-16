---
title: SprongBoot整合Security标签的使用
date: 2018-10-16 22:19:59
tags: [SpringBoot, Security]
---

# 引入pom.xml

```
 <!--引入thymeleaf与Spring Security整合的依赖-->
    <dependency>
       <groupId>org.thymeleaf.extras</groupId>
       <artifactId>thymeleaf-extras-springsecurity4</artifactId>
      <version>3.0.2.RELEASE</version>
    </dependency>
```

<!--more-->

# 在页面 引入
`xmlns:sec="http://www.thymeleaf.org/thymeleaf-extras-springsecurity4"`

# 使用

```
<span sec:authentication="name"></span>
sec:authentication="name"  获取用户名

sec:authentication="principal.authorities" 获取权限信息

sec:authorize="hasRole('ROLE_AAA') 判断角色

sec:authorize="hasAuthority('USER')" 判断权限

sec:authorize="hasAnyRole('USER','ROLE_AAA') 拥有角色之一

sec:authorize="!isAuthenticated()" 未登录

sec:authorize="isAuthenticated()" 已登录

```



# 例子

```
<div align="center" style="margin-top: 15px" sec:authorize="!isAuthenticated()">
    <h4 style="color: blue;">欢迎您，亲爱的召唤师!<a th:href="@{/login}"> 请登录</a></h4>
</div>

<div align="center" style="margin-top: 15px" sec:authorize="isAuthenticated()">
    <h4 style="color: blue;">召唤师 <span sec:authentication="name"></span>
        ! 您的段位为：<span sec:authentication="principal.authorities"></span>
    </h4>
    <form th:action="@{/logout}" method="post">
        <input type="submit" th:value="注销登录">
    </form>
</div>

<div align="center" style="margin-top: 100px" sec:authorize="hasRole('英勇青铜')">
    <a th:href="@{/bronze}">点击领取英勇青铜段位奖励</a>
</div>
<div align="center" style="margin-top: 100px" sec:authorize="hasRole('不屈白银')">
    <a th:href="@{/silver}">点击领取不屈白银段位奖励</a>
</div>
<div align="center" style="margin-top: 100px" sec:authorize="hasRole('荣耀黄金')">
    <a th:href="@{/gold}">点击领取荣耀黄金段位奖励</a>
</div>
<div align="center" style="margin-top: 100px" sec:authorize="hasRole('华贵铂金')">
    <a th:href="@{/platinum}">点击领取华贵铂金段位奖励</a>
</div>
<div align="center" style="margin-top: 100px" sec:authorize="hasRole('璀璨钻石')">
    <a th:href="@{/diamond}">点击领取璀璨钻石段位奖励</a>
</div>
<div align="center" style="margin-top: 100px" sec:authorize="hasRole('超凡大师')">
    <a th:href="@{/master}">点击领取超凡大师段位奖励</a>
</div>
<div align="center" style="margin-top: 100px" sec:authorize="hasRole('最强王者')">
    <a th:href="@{/challenger}">点击领取最强王者段位奖励</a>
</div>

```