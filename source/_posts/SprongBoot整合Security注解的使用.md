---
title: SprongBoot整合Security注解的使用
date: 2018-10-16 21:16:25
tags: [SpringBoot, Security]
---

>参考: [Spring Security——基于表达式的权限控制](https://blog.csdn.net/dsjnqq/article/details/52649366?utm_source=blogxgwz5)

# 注解的使用

在上文的`SecurityConfig`类中中加入

```
@EnableGlobalMethodSecurity(prePostEnabled = true)   
//开启Security注解  然后在controller中就可以使用方法注解

	/**
     * 防止注解使用不了
     */
    @Bean
    @Override
    public AuthenticationManager authenticationManagerBean() throws Exception {
        return super.authenticationManagerBean();
    }

```

<!--more-->

![](/img/2018-10-16/Sercurity3.png)


##  使用:

>需要注意的是 任何以ROLE_开头的权限都被视为角色

@EnableGlobalMethodSecurity详解

### 3.1、@EnableGlobalMethodSecurity(securedEnabled=true) 开启@Secured 注解过滤权限

### 3.2、@EnableGlobalMethodSecurity(jsr250Enabled=true)开启@RolesAllowed 注解过滤权限 

### 3.3、@EnableGlobalMethodSecurity(prePostEnabled=true) 使用表达式时间方法级别的安全性         4个注解可用

```
@PreAuthorize 在方法调用之前,基于表达式的计算结果来限制对方法的访问

@PostAuthorize 允许方法调用,但是如果表达式计算结果为false,将抛出一个安全性异常

@PostFilter 允许方法调用,但必须按照表达式来过滤方法的结果

@PreFilter 允许方法调用,但必须在进入方法之前过滤输入值

```

##具体使用:
### JSR-250注解:

```
 1.@DenyAll拒绝所有访问。

 2.@RolesAllowed({"USER","ADMIN"})该方法只要具有“USER","ADMIN"任意一种权限就可以访问。这里可以省略前缀ROLE_，实际的权限可能是ROLE_ADMIN

 3.@PermitAll允许所有访问

```
### prePostEnabled注解

`1. @PreAuthorize`  
在方法执行之前执行，而且这里可以调用方法的参数，也可以得到参数值，这里利用JAVA8的参数名反射特性，如果没有JAVA8，那么也可以利用Spring Secuirty的@P标注参数，或利用Spring Data的@Param标注参数

```
@PreAuthorize("#userId == authentication.principal.userId or hasAuthority(‘ADMIN’)")
 
void changePassword(@P("userId") long userId ){  }

```
还可以这种写法   

```
@PreAuthorize("hasRole('ROLE_AAA')")
@PreAuthorize("hasPermission('add')")
@PreAuthorize("hasAuthority('ROLE_AAA')")
@PreAuthorize("hasPermission('user', 'ROLE_USER')")


   @PreAuthorize("hasRole('ROLE_USER') or hasRole('ROLE_ADMIN')")
   public User find(int id) {

      System.out.println("find user by id............." + id);

      return null;

   }
}

       在上面的代码中我们定义了只有拥有角色ROLE_ADMIN的用户才能访问adduser()方法，而访问find()方法需要有ROLE_USER角色或ROLE_ADMIN角色。使用表达式时我们还可以在表达式中使用方法参数。下载 

public class UserServiceImpl implements UserService {

 

   /**

    * 限制只能查询Id小于10的用户

    */

   @PreAuthorize("#id<10")

   public User find(int id) {

      System.out.println("find user by id........." + id);

      return null;

   }

  

   /**

    * 限制只能查询自己的信息

    */

   @PreAuthorize("principal.username.equals(#username)")

   public User find(String username) {

      System.out.println("find user by username......" + username);

      return null;

   }

 

   /**

    * 限制只能新增用户名称为abc的用户

    */

   @PreAuthorize("#user.name.equals('abc')")

   public void add(User user) {

      System.out.println("addUser............" + user);

   }
}

```

>当@PreFilter标注的方法拥有多个集合类型的参数时，需要通过@PreFilter的filterTarget属性指定当前@PreFilter是针对哪个参数进行过滤的。如下面代码就通过filterTarget指定了当前@PreFilter是用来过滤参数ids的

```

@PreFilter(filterTarget="ids", value="filterObject%2==0")

   public void delete(List<Integer> ids, List<String> usernames) {

      ...

   }
   
```


---
---
---
