---
title: springmvc@requestBody注解的使用
date: 2018-11-09 15:19:18
tags: SpringMvc
---

# @requestBody注解的使用

* @requestBody注解常用来处理content-type不是默认的application/x-www-form-urlcoded编码的内容，比如说：application/json或者是application/xml等。一般情况下来说常用其来处理application/json类型。

<!--more-->

```
通过@requestBody可以将请求体中的JSON字符串绑定到相应的bean上，当然，也可以将其分别绑定到对应的字符串上。
　　　　例如说以下情况：
　　　　$.ajax({
　　　　　　　　url:"/login",
　　　　　　　　type:"POST",
　　　　　　　　data:'{"userName":"admin","pwd","admin123"}',
　　　　　　　　content-type:"application/json charset=utf-8",
　　　　　　　　success:function(data){
　　　　　　　　　　alert("request success ! ");
　　　　　　　　}
　　　　});
```

```
　@requestMapping("/login")
　　　　public void login(@RequestBody String userName,@requestBody String pwd){
　　　　　　System.out.println(userName+" ："+pwd);
　　　　}
　　　　这种情况是将JSON字符串中的两个变量的值分别赋予了两个字符串，但是呢假如我有一个User类，拥有如下字段：
　　　　　　String userName;
　　　　　　String pwd;
　　　　那么上述参数可以改为以下形式：@RequestBody User user 这种形式会将JSON字符串中的值赋予user中对应的属性上
　　　　需要注意的是，JSON字符串中的key必须对应user中的属性名，否则是请求不过去的。
```



```
　在一些特殊情况@RequestBody也可以用来处理content-type类型为application/x-www-form-urlcoded的内容，只不过这种方式

　　　　不是很常用，在处理这类请求的时候，@RequestBody会将处理结果放到一个MultiValueMap<String,String>中，这种情况一般在
　　　　特殊情况下才会使用，
　　　　例如jQuery easyUI的datagrid请求数据的时候需要使用到这种方式、小型项目只创建一个POJO类的话也可以使用这种接受方式
```

---


# 测试

```
    @RequestMapping(value = "/testRequestBody", method = RequestMethod.POST)
    @ResponseBody
    public String testRequestBody(@RequestBody UserModel userModel) {
        System.out.println("username is:" + userModel.getUsername());
        System.out.println("password is:" + userModel.getPassword());
        return "输入的userName:" + userModel.getUsername() + "输入的password:" + userModel.getPassword();
    }
```