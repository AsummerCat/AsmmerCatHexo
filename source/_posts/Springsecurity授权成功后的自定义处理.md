---
title: Springsecurity授权成功后的自定义处理
date: 2019-07-10 22:00:10
tags: [SpringCloud,Security,SpringBoot]
---

# Springsecurity授权成功后的自定义处理

## 成功后的处理 需要实现 AuthenticationSuccessHandler

## 失败后的处理 需要实现 AuthenticationFailureHandler



## 实现方式

这样做的好处就是 可以在处理用户登录权限的同时自定义一些自己需要的东西 比如记录日志啥的

<!--more-->

## 创建一个后置处理器

继承了默认的SimpleUrlAuthenticationSuccessHandler 实现了AuthenticationSuccessHandler

```
package com.linjingc.loginserversessiontoken.config.security;

import lombok.extern.slf4j.Slf4j;
import org.springframework.security.core.Authentication;
import org.springframework.security.web.authentication.AuthenticationSuccessHandler;
import org.springframework.security.web.authentication.SimpleUrlAuthenticationSuccessHandler;
import org.springframework.stereotype.Component;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

/**
 * 自定义登录成功 后置处理
 *
 * @author cxc
 * @date 2019/7/2 21:17
 */
@Slf4j
@Component
public class MyAuthenticationSuccessHandler extends SimpleUrlAuthenticationSuccessHandler implements AuthenticationSuccessHandler {
    @Override
    public void onAuthenticationSuccess(HttpServletRequest request,
                                        HttpServletResponse response, Authentication authentication)
            throws ServletException, IOException {
        //这里就可以写自定义的方法 比如创建日志
        System.out.println("创建日志");
        super.onAuthenticationSuccess(request, response, authentication);
    }
}

```

下一步如何添加到程序中

# 创建配置文件

```
package com.linjingc.loginserversessiontoken.config.security;

import com.linjingc.loginserversessiontoken.config.security.jwt.JwtUtils;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.method.configuration.EnableGlobalMethodSecurity;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.builders.WebSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.config.http.SessionCreationPolicy;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;

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

    @Autowired
    private MyAuthenticationSuccessHandler myAuthenticationSuccessHandler;

    /**
     * 这里可以设置忽略的路径或者文件
     */
    @Override
    public void configure(WebSecurity web) {
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


        // 开启session
        http.sessionManagement().sessionCreationPolicy(SessionCreationPolicy.IF_REQUIRED).maximumSessions(1);

        // 去掉 CSRF
        http.csrf().disable()
                .authorizeRequests()
                //不需要权限访问
                .antMatchers("/**.html", "/**.html", "/**.css", "/img/**", "/**.js", "/third-party/**").permitAll()
                //该路径需要验证通过
                .antMatchers("/", "/index/", "/index/**").authenticated()
                //该路径需要角色  or 权限XXX
                .antMatchers("/lo").access("hasRole('AAA') or hasAuthority('add')")
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
                .successHandler(myAuthenticationSuccessHandler)
                //自定义登录页
                .loginPage("/login")
//                //登录成功页面
                .defaultSuccessUrl("/hello")
                .permitAll()
                .and()
                //添加jwt验证
                //权限校验
                .addFilter((new JWTAuthorizationFilter(authenticationManager(), jwtUtils)))
                .logout().deleteCookies("login-session")
                //退出登录后的默认url是"/home"
                .logoutSuccessUrl("/byeBye");

        // http.addFilterBefore(new MyAuthenticationFilter(authenticationManager(), jwtConfig, jwtUtils), UsernamePasswordAuthenticationFilter.class);

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
        //在此处应用自定义PasswordEncoder
        auth.inMemoryAuthentication().passwordEncoder(bCryptPasswordEncoder())
                //写在内存中的角色 密码heihei
                .withUser("user").password("$2a$10$rqOD.4PCiTJUm3BrNDjxfO287rWocQCjT7p/TE3YwTi6LhSXSX0Ba").roles("USER")
                .and() //这个是指可以写多个
                .withUser("admin").password("$2a$10$rqOD.4PCiTJUm3BrNDjxfO287rWocQCjT7p/TE3YwTi6LhSXSX0Ba").authorities("ROLE_USER", "ROLE_ADMIN");

        /**
         * 方式二 数据库查询用户信息
         */
        //     auth.userDetailsService(customUserService()).passwordEncoder(bCryptPasswordEncoder());//添加自定义的userDetailsService认证  //现在已经要加密.passwordEncoder(new MyPasswordEncoder())
        //    auth.eraseCredentials(false);   //这里是清除还是不清除登录的密码  SecurityContextHolder中
    }

    /**
     * BCryptPasswordEncoder 使用BCrypt的强散列哈希加密实现，并可以由客户端指定加密的强度strength，强度越高安全性自然就越高，默认为10.
     * 自定义密码加密器
     * BCryptPasswordEncoder(int strength, SecureRandom random)
     * SecureRandom secureRandom3 = SecureRandom.getInstance("SHA1PRNG");
     */
    @Bean
    public static BCryptPasswordEncoder bCryptPasswordEncoder() {
        return new BCryptPasswordEncoder();
    }

    /**
     * 防止注解使用不了
     */
    @Bean
    @Override
    public AuthenticationManager authenticationManagerBean() throws Exception {

        return super.authenticationManagerBean();
    }

}

```





两种方式

## 方式一

创建一个自定义的登录授权过滤器 注入自定义的handler

` setAuthenticationSuccessHandler(new MyAuthenticationSuccessHandler());`

```
package com.linjingc.loginserversessiontoken.config.security;

import com.linjingc.loginserversessiontoken.config.security.jwt.JwtUtils;
import com.linjingc.loginserversessiontoken.entity.BasicUser;
import lombok.extern.slf4j.Slf4j;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.AuthenticationException;
import org.springframework.security.web.authentication.AbstractAuthenticationProcessingFilter;
import org.springframework.security.web.util.matcher.AntPathRequestMatcher;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.util.ArrayList;

/**
 * JWT 整合Spring Security 登录密码校验类
 * 这里开启后 SecurityConfig中 需要移除  .successHandler(myAuthenticationSuccessHandler)
 * 这里开启后 SecurityConfig中 需要添加   http.addFilterBefore(new MyAuthenticationFilter(authenticationManager(), jwtConfig, jwtUtils), UsernamePasswordAuthenticationFilter.class);
 *
 * @author cxc
 * @date 2019年6月27日13:08:02
 */
@Slf4j
public class MyAuthenticationFilter extends AbstractAuthenticationProcessingFilter {

    private AuthenticationManager authenticationManager;
    private JwtUtils jwtUtils;

    public MyAuthenticationFilter(AuthenticationManager authenticationManager, JwtUtils jwtUtils) {

        super(new AntPathRequestMatcher("/login", "POST"));
        this.authenticationManager = authenticationManager;
        this.jwtUtils = jwtUtils;

    }

    @Override
    public Authentication attemptAuthentication(HttpServletRequest request, HttpServletResponse response) throws AuthenticationException {
        String username = request.getParameter("username");
        String password = request.getParameter("password");
        if (username == null) {
            username = "";
        }
        if (password == null) {
            password = "";
        }
        BasicUser loginUser = new BasicUser();
        loginUser.setUsername(username);
        loginUser.setPassword(password);
        UsernamePasswordAuthenticationToken authRequest = new UsernamePasswordAuthenticationToken(loginUser.getUsername(), loginUser.getPassword(), new ArrayList<>());
        Authentication authentication = this.authenticationManager.authenticate(authRequest);
        if (authentication != null) {
            super.setContinueChainBeforeSuccessfulAuthentication(true);
            //因为是使用seesion管理的话 这边使用自定义注册token
            // setSessionAuthenticationStrategy(new RegisterTokenAuthenticationStrategy(jwtUtils));
            setAuthenticationSuccessHandler(new MyAuthenticationSuccessHandler());
        }
        return authentication;
    }
}

```

### 下一步修改配置类 添加这个过滤器到配置文件中

```
         http.addFilterBefore(new MyAuthenticationFilter(authenticationManager(), jwtConfig, jwtUtils), UsernamePasswordAuthenticationFilter.class);

```

## 方式二

```
直接在配置类上写入
 @Autowired
    private MyAuthenticationSuccessHandler myAuthenticationSuccessHandler;


  .successHandler(myAuthenticationSuccessHandler)
```

