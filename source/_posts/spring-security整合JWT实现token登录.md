---
title: spring-security整合JWT实现token登录
date: 2019-07-10 21:56:21
tags: [Security,JWT,springCloud]
---

# spring-security整合JWT实现token登录

[demo地址](https://github.com/AsummerCat/springcloudAll/tree/master/loginServer-token)  demo中的loginServer-token模块

# SpirngBoot 和cloud版本

```java
  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.1.6.RELEASE</version>
    <relativePath/> <!-- lookup parent from repository -->
  </parent>
    <spring-cloud.version>Greenwich.SR1</spring-cloud.version>
```

<!--more-->

# 导入pom文件

```java
    <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-security</artifactId>
    </dependency>
     <!--JWT -->
        <dependency>
            <groupId>com.auth0</groupId>
            <artifactId>java-jwt</artifactId>
            <version>3.8.1</version>
        </dependency>
        
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.31</version>
        </dependency>

  <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
```

# JWT 整合Spring Security 密码校验类

```java
package com.linjingc.loginservertoken.config.security;

import com.alibaba.fastjson.JSON;
import com.linjingc.loginservertoken.config.jwt.JwtUtils;
import com.linjingc.loginservertoken.entity.BasicUser;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.authentication.AuthenticationServiceException;
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.AuthenticationException;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter;

import javax.servlet.FilterChain;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.util.ArrayList;
import java.util.UUID;

/**
 * JWT 整合Spring Security 密码校验类
 *
 * @author cxc
 * @date 2019年6月27日13:08:02
 */
public class JWTAuthenticationFilter extends UsernamePasswordAuthenticationFilter {

    private AuthenticationManager authenticationManager;

    private JwtUtils jwtUtils;

    JWTAuthenticationFilter(AuthenticationManager authenticationManager, JwtUtils jwtUtils) {
        this.authenticationManager = authenticationManager;
        this.jwtUtils = jwtUtils;
    }


    @Override
    public Authentication attemptAuthentication(HttpServletRequest request,
                                                HttpServletResponse response) throws AuthenticationException {

        //限制只有post请求才能进入登录操作
        if (!request.getMethod().equals("POST")) {
            throw new AuthenticationServiceException("认证方法请使用POST请求: " + request.getMethod());
        } else {
            String username = this.obtainUsername(request);
            String password = this.obtainPassword(request);
            if (username == null) {
                username = "";
            }
            if (password == null) {
                password = "";
            }
            try {
                BasicUser loginUser = new BasicUser();
                loginUser.setUsername(username);
                loginUser.setPassword(password);
                UsernamePasswordAuthenticationToken authRequest = new UsernamePasswordAuthenticationToken(loginUser.getUsername(), loginUser.getPassword(), new ArrayList<>());
                this.setDetails(request, authRequest);
                return authenticationManager.authenticate(authRequest);
            } catch (Exception e) {
                e.printStackTrace();
                return null;
            }
        }

    }


    // 成功验证后调用的方法
    // 如果验证成功，就生成token并返回
    @Override
    protected void successfulAuthentication(HttpServletRequest request,
                                            HttpServletResponse response,
                                            FilterChain chain,
                                            Authentication authResult) throws IOException, ServletException {

        // 查看源代码会发现调用getPrincipal()方法会返回一个实现了`UserDetails`接口的对象
        UserDetails userDetails = (UserDetails) authResult.getPrincipal();

        //签发token
        String token = jwtUtils.createJWT(UUID.randomUUID().toString(), JSON.toJSONString(userDetails), userDetails.getUsername());
        // 但是这里创建的token只是单纯的token
        // 按照jwt的规定，最后请求的格式应该是 `Bearer token`
        response.setHeader(jwtUtils.tokenHeader, jwtUtils.tokenPrefix + token);
        //登录成功后转发到首页
        request.getRequestDispatcher("/").forward(request, response);
    }


    /**
     * 这是验证失败时候调用的方法
     *
     * @param request
     * @param response
     * @param failed
     * @throws IOException
     * @throws ServletException
     */
    @Override
    protected void unsuccessfulAuthentication(HttpServletRequest request, HttpServletResponse response, AuthenticationException failed) throws IOException, ServletException {
        response.getWriter().write("authentication failed, reason: " + failed.getMessage());
    }
}

```



# * JWT 整合Spring Security* 访问认证类

```
package com.linjingc.loginservertoken.config.security;

import com.linjingc.loginservertoken.config.jwt.JwtUtils;
import org.apache.commons.lang.StringUtils;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.security.web.authentication.www.BasicAuthenticationFilter;

import javax.servlet.FilterChain;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.util.Objects;

/**
 * JWT 整合Spring Security
 * 访问认证类
 *
 * @author cxc
 * @date 2019年6月27日15:09:19
 */
public class JWTAuthorizationFilter extends BasicAuthenticationFilter {

    private JwtUtils jwtUtils;

    JWTAuthorizationFilter(AuthenticationManager authenticationManager, JwtUtils jwtUtils) {
        super(authenticationManager);
        this.jwtUtils = jwtUtils;
    }

    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                    HttpServletResponse response,
                                    FilterChain chain) throws IOException, ServletException {

        String tokenHeader = request.getHeader(jwtUtils.tokenHeader);
        // 如果请求头中没有Authorization信息则直接放行了
        if (StringUtils.isEmpty(tokenHeader) || !tokenHeader.startsWith(jwtUtils.tokenPrefix)) {
            chain.doFilter(request, response);
            return;
        }
        // 如果请求头中有token，则进行解析，并且设置认证信息
        try {
            //fixme 这里有个问题不应该把用户的权限信息也赋值进去 传输给页面的token
            SecurityContextHolder.getContext().setAuthentication(getAuthentication(tokenHeader));
        } catch (UsernameNotFoundException e) {
//           throw new ServletException("无法获取用户信息");
            onUnsuccessfulAuthentication(request, response, e);
            return;
        }


        super.doFilterInternal(request, response, chain);
    }

    /**
     * 这里从token中获取用户信息并新建一个token
     *
     * @param tokenHeader
     * @return
     */
    private UsernamePasswordAuthenticationToken getAuthentication(String tokenHeader) throws UsernameNotFoundException {
        String token = tokenHeader.replace(jwtUtils.tokenPrefix, "");
        try {
            UserDetails user = jwtUtils.getUser(token);
            if (Objects.nonNull(user) && StringUtils.isNotEmpty(user.getUsername())) {
                return new UsernamePasswordAuthenticationToken(user.getUsername(), null, user.getAuthorities());
            }
        } catch (Exception e) {
            throw new UsernameNotFoundException("无法获取用户信息");
        }
        return null;
    }
}

```

# 实现JWT工具类

```java
package com.linjingc.loginservertoken.config.jwt;

import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.JSONObject;
import com.auth0.jwt.JWT;
import com.auth0.jwt.JWTVerifier;
import com.auth0.jwt.algorithms.Algorithm;
import com.auth0.jwt.interfaces.DecodedJWT;
import com.linjingc.loginservertoken.entity.BasicUser;
import lombok.Data;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.stereotype.Component;

import java.util.Date;
import java.util.HashMap;
import java.util.Map;

@Component
@Data
public class JwtUtils {
    private Algorithm algorithm;
    @Value("${basic.jwt.secret}")
    private String secret;
    @Value("${basic.jwt.tokenPrefix}")
    public String tokenPrefix;
    @Value("${basic.jwt.tokenHeader}")
    public String tokenHeader;
    @Value("${basic.jwt.issuer}")
    public String issuer;
    /**
     * 有效时间
     */
    @Value("${basic.jwt.expire}")
    private long expire;


    /**
     * 创建JWT签名
     *
     * @param id
     * @param subject  主题 可以是JSON数据 尽可能少
     * @param audience 签名接受者
     * @return
     */
    public String createJWT(String id, String subject, String audience) {
        Algorithm algorithm = Algorithm.HMAC256(secret);
        Map<String, Object> map = new HashMap<String, Object>(4);
        map.put("alg", "HS256");
        map.put("typ", "JWT");

        Date nowDate = new Date();
        // 过期时间
        Date expireDate = new Date(nowDate.getTime() + expire * 1000);

        String token = JWT.create()
                //header
                .withHeader(map)
                /*设置 载荷 Payload*/
                .withClaim("org", "www.linjingc.top")
                //签名是有谁生成 例如 服务器
                .withIssuer(issuer)
                //签名的主题
                .withSubject(subject)
                //.withNotBefore(new Date())//该jwt都是不可用的时间
                //签名的观众 也可以理解谁接受签名的
                .withAudience(audience)
                //生成签名的时间
                .withIssuedAt(nowDate)
                //签名过期的时间
                .withExpiresAt(expireDate)
                /*签名 Signature */
                .sign(algorithm);
        return token;
    }

    /**
     * 测试方法
     *
     * @param arr
     */
    public static void main(String[] arr) {
        String jwt = new JwtUtils().createJWT("no.1", "这是个json消息", "夏天的猫");
    }

    /**
     * 从token中获取数据
     *
     * @param token
     * @return
     */
    public UserDetails getUser(String token) {
        Algorithm algorithm = Algorithm.HMAC256(secret);
        JWTVerifier verifier = JWT.require(algorithm).withIssuer(issuer).build();
        DecodedJWT jwt = verifier.verify(token);

        JSONObject userJson = JSONObject.parseObject(jwt.getSubject());
        BasicUser subject = JSON.toJavaObject(userJson, BasicUser.class);
        return subject;
    }
}

```



# Security配置类

```
package com.linjingc.loginservertoken.config.security;

import com.linjingc.loginservertoken.config.jwt.JwtUtils;
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
    private JwtUtils jwtUtils;

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


        //关闭session
        http.sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS);

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
                //.rememberMe()
                //设置cookie有效期
                //.tokenValiditySeconds(60 * 60 * 24 * 7)
                //.and()
                .formLogin()
                //自定义登录页
                .loginPage("/login")
//                //登录成功页面
                .defaultSuccessUrl("/hello")
                .permitAll()
                .and()
                //添加jwt验证
                //登录校验
                .addFilter((new JWTAuthenticationFilter(authenticationManager(), jwtUtils)))
                //权限校验
                .addFilter((new JWTAuthorizationFilter(authenticationManager(), jwtUtils)))
                .logout()
                //退出登录后的默认url是"/home"
                .logoutSuccessUrl("/byeBye");
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



# 实体类

```java
package com.linjingc.loginservertoken.entity;

import lombok.Data;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.authority.SimpleGrantedAuthority;
import org.springframework.security.core.userdetails.UserDetails;

import java.util.Collection;
import java.util.List;

@Data
public class BasicUser implements UserDetails {
    private long id;
    private String username;
    private String password;
    private List<SimpleGrantedAuthority> authorities;

    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return null;
    }

    @Override
    public boolean isAccountNonExpired() {
        return false;
    }

    @Override
    public boolean isAccountNonLocked() {
        return false;
    }

    @Override
    public boolean isCredentialsNonExpired() {
        return false;
    }

    @Override
    public boolean isEnabled() {
        return false;
    }
}

```



# 配置文件

```java
spring:
  application:
    name: Login-Server-token

server:
  port: 8500


## eureka 客户端基本配置
eureka:
  instance:
    prefer-ip-address: true
    instance-id: ${spring.cloud.client.ip-address}:${server.port}
  client:
    serviceUrl:
      defaultZone: http://admin:pass@localhost:8100/eureka
    healthcheck:
      enabled: true




basic:
  #JWT配置
  jwt:
    ## header:凭证(校验的变量名), expire:有效期1天(单位:s), secret:秘钥(普通字符串)
    secret: secret
    tokenPrefix: Bearer
    tokenHeader: Authorization
    issuer: issuer
    expire: 5184000
```

