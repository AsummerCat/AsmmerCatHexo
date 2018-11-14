---
title: springmvc请求参数获取的几种方法
date: 2018-11-09 13:55:51
tags: SpringMvc
---

# 直接把表单的参数写在Controller相应的方法的形参中，适用于get方式提交，不适用于post方式提交。

<!--more-->

```
/**
     * 1.直接把表单的参数写在Controller相应的方法的形参中
      * @param username
     * @param password
     * @return
     */
    @RequestMapping("/addUser1")
    public String addUser1(String username,String password) {
        System.out.println("username is:"+username);
        System.out.println("password is:"+password);
        return "demo/index";
    }
```
>http://localhost:8080/testGet?username=小明&password=10086


---

# 通过HttpServletRequest接收，post方式和get方式都可以。

```
 @RequestMapping(value = "testRequest")
    public String testRequest(HttpServletRequest request) {
        String username = request.getParameter("username");
        String password = request.getParameter("password");
        return "输入的userName:" + username + "输入的password:" + password;
    }

```

---

# 通过一个bean来接收,post方式和get方式都可以。

实体类 

```
package com.linjingc.demo;

import lombok.Data;
import lombok.ToString;

@Data
@ToString
public class UserModel {

    private String username;
    private String password;
}
```

```
  /**
     * 通过一个bean来接收,post方式和get方式都可以。
     */
    @RequestMapping(value = "testBean")
    public String testBean(UserModel userModel) {
        return "输入的userName:" + userModel.getUsername() + "输入的password:" + userModel.getPassword();
    }
```

# 通过@PathVariable获取路径中的参数

```
  /**
     * 通过@PathVariable获取路径中的参数
     */
    @GetMapping(value = "testPathVariable/{user}/{password}")
    public String testPathVariable(@PathVariable(name = "user") String userName, @PathVariable String password) {
        return "输入的userName:" + userName + "输入的password:" + password;
    }
```
<font colo="red">需要注意的是 如果路径不写完整会报 404  
所以在不需要的时候添加</font>

http://localhost:8080/testPathVariable/user/password

# 使用@ModelAttribute注解获取POST请求的FORM表单数据

Jsp表单如下：

```
<form action ="<%=request.getContextPath()%>/demo/addUser5" method="post"> 
     用户名:&nbsp;<input type="text" name="username"/><br/>
     密&nbsp;&nbsp;码:&nbsp;<input type="password" name="password"/><br/>
     <input type="submit" value="提交"/> 
     <input type="reset" value="重置"/> 
</form>
```

```
/**
     * 5、使用@ModelAttribute注解获取POST请求的FORM表单数据
      * @param user
     * @return
     */
    @RequestMapping(value="/addUser5",method=RequestMethod.POST)
    public String addUser5(@ModelAttribute("user") UserModel user) {
        System.out.println("username is:"+user.getUsername());
        System.out.println("password is:"+user.getPassword());
        return "demo/index";
    }
```

# 用注解@RequestParam绑定请求参数到方法入参

当请求参数username不存在时会有异常发生,可以通过设置属性required=false解决,例如: @RequestParam(value="username", required=false)

```
  /**
     * 用注解@RequestParam绑定请求参数到方法入参
     * 当请求参数username不存在时会有异常发生,可以通过设置属性required=false解决,例如: @RequestParam(value="username", required=false)
     */
    @RequestMapping(value = "/addUser", method = RequestMethod.GET)
    public String addUser6(@RequestParam(value = "username", required = false) String userName, @RequestParam("password") String password) {
        System.out.println("username is:" + userName);
        System.out.println("password is:" + password);
        return "输入的userName:" + userName + "输入的password:" + password;
    }
```
>http://localhost:8080/addUser?password=1&&username=小明
>
>http://localhost:8080/addUser?password=1