---
title: SprongBoot整合Security
date: 2018-10-16 15:17:54
tags: [SpringBoot, Security]
---
# [demo地址](https://github.com/AsummerCat/SpringBootAndSecurity)

---

# 导入相关jar包

```
<!--security权限控制-->
       <dependency>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-starter-security</artifactId>
       </dependency>
       
<!--thymeleaf页面模板-->
		<dependency>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-starter-thymeleaf</artifactId>
       </dependency>
       <!--引入thymeleaf与Spring Security整合的依赖-->
    <dependency>
       <groupId>org.thymeleaf.extras</groupId>
       <artifactId>thymeleaf-extras-springsecurity4</artifactId>
      <version>3.0.2.RELEASE</version>
    </dependency>
```
<!--more-->

---

#  直接运行SpringBoot

```
那么你会发现 在控制台 会出现
Using generated security password: 07dc7832-2928-4eea-8329-b5fecdd51d57

这里是集成Security后会出现的name为User 密码为:07dc7832-2928-4eea-8329-b5fecdd51d57的一个默认用户

```
---

# 自定义SecurityConfig类

`继承WebSecurityConfigurerAdapter`   
重写`configure`方法  
并且并加上`@Configuration`   
和`@EnableWebSecurity `
`@EnableGlobalMethodSecurity(prePostEnabled=true) `  开启Security注解  然后在controller中就可以使用方法注解
3个注解。


![](/img/2018-10-16/security1.png)

---

# 在配置文件需要重写三个方法以实现自定义的权限校验管理

![](/img/2018-10-16/security2.png)

## 1.静态页面忽略
重写:  
`@Override public void configure(WebSecurity web)`

## 2.权限管理
重写:
` protected void configure(HttpSecurity http) throws Exception {`

## 3.验证登录及其权限赋予
重写: 
`public void configure(AuthenticationManagerBuilder auth) throws Exception {`

---

# 创建一个loginController
 略
 
---
# 创建一个User类
 略
 
---

# 利用thymeleaf创建一个登陆页面

```
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml"
      xmlns:th="http://www.thymeleaf.org" xmlns:sec="http://www.thymeleaf.org/thymeleaf-extras-springsecurity3">
<head>
    <title>Spring Security Example </title>
</head>
<body>
<div th:if="${param.error}">
    用户名或密码错
</div>
<div th:if="${param.logout}">
    您已注销成功
</div>
<form th:action="@{/login}" method="post">
    <div><label> 用户名 : <input type="text" name="username"/> </label></div>
    <div><label> 密  码 : <input type="password" name="password"/> </label></div>    
    <div><label> 记住我 : <input id="remember_me" name="remember-me" type="checkbox"/></label></div>
    <div><input type="submit" value="登录"/></div>
</form>
</body>
</html>
```
---

# 获取登录后的用户信息

**<font color="red">这里说明一下:这里最好是写一个自己的User类继承UserDetails 这样就可以获取更多的属性了</font>**

```
 UserDetails userDetails = (UserDetails) SecurityContextHolder.getContext().getAuthentication() .getPrincipal();   //登陆后的账号 转换为 UserDetails
 
 accountnonexpired=true  代表当前这个账户不过期
 accountNonLocked=true   代表当前账户未锁定
 credentialsNonExpired=true 代表当前账户凭证未过期
 authorities             用户的权限集合set
 enabled=true            代表当前账户可用
```

---

# 创建自定义凭证匹配器(密码校验器)
 需要实现 `PasswordEncoder` 接口
 需要注意的是 如果没有配置这个的话 在下面重写验证登录的方法 会报错 :
 `java.lang.IllegalArgumentException: There is no PasswordEncoder mapped for the id "null"` 
 
 有三种实现方式: 
## 1.继承 NoOpPasswordEncoder 啥也不做按原文本处理，相当于不加密

```
 /**
     * 不加密 官方已经不推荐了
     * 自定义密码加密器
     */
    @Bean
    public static NoOpPasswordEncoder passwordEncoder() {
        return (NoOpPasswordEncoder) NoOpPasswordEncoder.getInstance();
    }

```

## 2.继承 StandardPasswordEncoder
>StandardPasswordEncoder 1024次迭代的SHA-256散列哈希加密实现，并使用一个随机8字节的salt。

```
  /**
     *StandardPasswordEncoder 1024次迭代的SHA-256散列哈希加密实现，并使用一个随机8字节的salt。
     * 自定义密码加密器
     * 盐值不需要用户提供，每次随机生成；
     *public StandardPasswordEncoder(CharSequence secret) 可以设置一个秘钥值
     * 计算方式: 迭代SHA算法+密钥+随机盐来对密码加密，加密后得到的密码是80位
     */
    @Bean
    public static StandardPasswordEncoder sCryptPasswordEncoder() {
        return new StandardPasswordEncoder();
    }


```

## 3. 继承 BCryptPasswordEncoder
>BCryptPasswordEncoder 使用BCrypt的强散列哈希加密实现，并可以由客户端指定加密的强度strength，强度越高安全性自然就越高，默认为10.

**<font color="red">在Spring的注释中，明确写明了如果是开发一个新的项目，BCryptPasswordEncoder是较好的选择。</font>**

```
  /**
     *BCryptPasswordEncoder 使用BCrypt的强散列哈希加密实现，并可以由客户端指定加密的强度strength，强度越高安全性自然就越高，默认为10.
     * 自定义密码加密器
     *BCryptPasswordEncoder(int strength, SecureRandom random)
     * SecureRandom secureRandom3 = SecureRandom.getInstance("SHA1PRNG");
     */
    @Bean
    public static BCryptPasswordEncoder bCryptPasswordEncoder() {
        return new BCryptPasswordEncoder();
    }
```

---

# 在SecurityConfig类中 重写验证登录和添加权限的方法

这里的话 有两种实现方式  
1.使用内存中账号和密码  
2.使用数据库查询用户信息

```
第一种:  
/**
         * 方式一 写在内存中的角色
         */
        auth.inMemoryAuthentication().passwordEncoder(bCryptPasswordEncoder())//在此处应用自定义PasswordEncoder
                .withUser("user").password("password").roles("USER")  //写在内存中的角色
                .and() //这个是指可以写多个
                .withUser("admin").password("password").authorities("ROLE_USER","ROLE_ADMIN");

```

```
第二种:  

         /**
         * 方式二 数据库查询用户信息
         */
        auth.userDetailsService(customUserService()).passwordEncoder(bCryptPasswordEncoder());//添加自定义的userDetailsService认证  //现在已经要加密.passwordEncoder(new MyPasswordEncoder())
        auth.eraseCredentials(false);   //这里是清除还是不清除登录的密码  SecurityContextHolder中
        
```

运行到这里基本上就构建完成了 

---

# 在SecurityConfig类中 重写权限控制

```
  /**
     * 这里是权限控制配置
     */
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        log.info("--------------------------SecurityConfig加载成功----------------------------");
        http.authorizeRequests()
                .antMatchers("/home", "/**.html", "/**.html", "/**.css", "/img/**", "/**.js", "/third-party/**").permitAll() //不需要权限访问
                .antMatchers("/", "/index/", "/index/**").authenticated()   //该路径需要验证通过
                .antMatchers("/lo").access("hasRole('AAA') or hasAuthority('add')") //该路径需要角色  or 权限XXX
                //还有一种方法 就是在方法名称上写注解    @PreAuthorize("hasAnyAuthority('USER','delete')")   //注解拦截权限
                //任何以ROLE_开头的权限都被视为角色
                .anyRequest().authenticated() //都要权限  放在最后
                .and()
                //开启cookie保存用户数据
                .rememberMe()
                //设置cookie有效期
                .tokenValiditySeconds(60 * 60 * 24 * 7)
                .and()
                .formLogin()
                .loginPage("/login")   //自定义登录页
                .usernameParameter("username")//自定义用户名参数名称
                .passwordParameter("password")//自定义密码参数名称
                .permitAll()
                .and()
                .logout()
                .logoutSuccessUrl("/home")//退出登录后的默认url是"/home"
                .permitAll();
    }
```


---


# 使用数据库查询用户

 需要实现 `UserDetailsService` 用户验证接口
 重写  
 `public UserDetails loadUserByUsername(String userName) throws UsernameNotFoundException { `方法
 
 ```
 
 package com.linjingc.demo.securityconfig;

import org.springframework.security.core.authority.SimpleGrantedAuthority;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.core.userdetails.UsernameNotFoundException;

import java.util.ArrayList;
import java.util.List;


/**
 * 自定义查询用户    Security接口
 */
public class CustomUserService implements UserDetailsService { //实现 这个用户验证接口
    @Override
    public UserDetails loadUserByUsername(String userName) throws UsernameNotFoundException {   //这里重构了原来的的方法
        //User user = userRepository.findByUsername(s); //jpa 查询数据库
        User user = new User();
        user.setUsername("ad");    //模拟查询
        user.setPassword("123456");


        List<SimpleGrantedAuthority> auths = new ArrayList<>();  //权限
        auths.add(new SimpleGrantedAuthority("add"));//这里添加权限 可以添加多个权限
        auths.add(new SimpleGrantedAuthority("update"));
        auths.add(new SimpleGrantedAuthority("delete"));
        auths.add(new SimpleGrantedAuthority("USER"));
        auths.add(new SimpleGrantedAuthority("ROLE_AAA"));  //任何以ROLE_开头的权限都被视为角色


        if (user == null) {
            throw new UsernameNotFoundException("用户名不存在");
        }
        System.out.println("userName:" + userName);
        System.out.println("username:" + user.getUsername() + ";password:" + user.getPassword());
        return new org.springframework.security.core.userdetails.User(user.getUsername(),
                user.getPassword(), auths);
    }
}
 
 ```
 ---
 
# 启动记住我功能

只要在configure上加入 `.rememberMe()`即可  

```
例如:  
@Override
protected void configure(HttpSecurity http) throws Exception{
    http
      .formLogin()
        .loginPage("/login")
      .and()
      .rememberMe()
         .tokenValiditySeconds(2419200)
         .key("spittrKey")
  ...
}
```



登录页上加入:

`记住我 : <input id="remember_me" name="remember-me" type="checkbox"/>`

这里需要注意的是:   
 默认情况下，这个功能是通过在Cookie中存储一个token完成的，这个token最多两周内有效。但是，在这里，我们指定这个token最多四周内有效（2,419,200秒）。存储在cookie中的token包含用户名、密码、过期时间和一个私匙——在写入cookie前都进行了MD5哈希。默认情况下，私匙的名为SpringSecured，但是这里我们将其设置为spitterKey，使他专门用于Spittr应用。

 
 ---
 
# 自定义SecurityConfig类(完整)
 
```
package com.linjingc.demo.securityconfig;

import com.linjingc.demo.service.CustomUserService;
import lombok.extern.slf4j.Slf4j;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.method.configuration.EnableGlobalMethodSecurity;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.builders.WebSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.NoOpPasswordEncoder;
import org.springframework.security.crypto.password.StandardPasswordEncoder;

/**
 * @author cxc
 * @date 2018/10/16 15:36
 * Security配置类
 */
@Slf4j
@Configuration
@EnableWebSecurity //注解开启Security
@EnableGlobalMethodSecurity(prePostEnabled = true)   //开启Security注解  然后在controller中就可以使用方法注解
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Bean
    UserDetailsService customUserService() {
        return new CustomUserService();
    }

    /**
     * 这里可以设置忽略的路径或者文件
     */
    @Override
    public void configure(WebSecurity web) throws Exception {
        //忽略css.jq.img等文件
        log.info("--------------------------SecurityConfig忽略文件及路径----------------------------");
        web.ignoring().antMatchers("/**.html", "/**.css", "/img/**", "/**.js", "/third-party/**");
    }


    /**
     * 这里是权限控制配置
     */
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        log.info("--------------------------SecurityConfig加载成功----------------------------");
        http.authorizeRequests()
                .antMatchers("/home", "/**.html", "/**.html", "/**.css", "/img/**", "/**.js", "/third-party/**").permitAll() //不需要权限访问
                .antMatchers("/", "/index/", "/index/**").authenticated()   //该路径需要验证通过
                .antMatchers("/lo").access("hasRole('AAA') or hasAuthority('add')") //该路径需要角色  or 权限XXX
                //还有一种方法 就是在方法名称上写注解    @PreAuthorize("hasAnyAuthority('USER','delete')")   //注解拦截权限
                //任何以ROLE_开头的权限都被视为角色
                .anyRequest().authenticated() //都要权限  放在最后
                .and()
                //开启cookie保存用户数据
                .rememberMe()
                //设置cookie有效期
                .tokenValiditySeconds(60 * 60 * 24 * 7)
                .and()
                .formLogin()
                .loginPage("/login")   //自定义登录页
                .usernameParameter("username")//自定义用户名参数名称
                .passwordParameter("password")//自定义密码参数名称
                .permitAll()
                .and()
                .logout()
                .logoutSuccessUrl("/home")//退出登录后的默认url是"/home"
                .permitAll();
    }


    /**
     * 这里是验证登录并且赋予权限
     *
     * @param auth
     * @throws Exception
     */
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        log.info("--------------------------Security自定义验证登录赋予权限方法加载成功----------------------------");

        /**
         * 方式一 写在内存中的角色
         */
        auth.inMemoryAuthentication().passwordEncoder(bCryptPasswordEncoder())//在此处应用自定义PasswordEncoder
                .withUser("user").password("password").roles("USER")  //写在内存中的角色
                .and() //这个是指可以写多个
                .withUser("admin").password("password").authorities("ROLE_USER", "ROLE_ADMIN");

        /**
         * 方式二 数据库查询用户信息
         */
        auth.userDetailsService(customUserService()).passwordEncoder(bCryptPasswordEncoder());//添加自定义的userDetailsService认证  //现在已经要加密.passwordEncoder(new MyPasswordEncoder())
        auth.eraseCredentials(false);   //这里是清除还是不清除登录的密码  SecurityContextHolder中
    }



    /**
     * 不加密 官方已经不推荐了
     * 自定义密码加密器
     */
    @Bean
    public static NoOpPasswordEncoder passwordEncoder() {
        return (NoOpPasswordEncoder) NoOpPasswordEncoder.getInstance();
    }

    /**
     *BCryptPasswordEncoder 使用BCrypt的强散列哈希加密实现，并可以由客户端指定加密的强度strength，强度越高安全性自然就越高，默认为10.
     * 自定义密码加密器
     *BCryptPasswordEncoder(int strength, SecureRandom random)
     * SecureRandom secureRandom3 = SecureRandom.getInstance("SHA1PRNG");
     */
    @Bean
    public static BCryptPasswordEncoder bCryptPasswordEncoder() {
        return new BCryptPasswordEncoder();
    }

    /**
     *StandardPasswordEncoder 1024次迭代的SHA-256散列哈希加密实现，并使用一个随机8字节的salt。
     * 自定义密码加密器
     * 盐值不需要用户提供，每次随机生成；
     *public StandardPasswordEncoder(CharSequence secret) 可以设置一个秘钥值
     * 计算方式: 迭代SHA算法+密钥+随机盐来对密码加密，加密后得到的密码是80位
     */
    @Bean
    public static StandardPasswordEncoder standardPasswordEncoder() {
        return new StandardPasswordEncoder();
    }

/**
	*防止注解使用不了
	*/
 @Bean
    @Override
    public AuthenticationManager authenticationManagerBean() throws Exception {
        return super.authenticationManagerBean();
    }

}

```  