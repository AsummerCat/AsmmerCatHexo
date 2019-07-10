---
title: SpringSecurity整合Oauth2构建基本可用的服务二
date: 2019-07-10 22:03:57
tags: [Security,Oauth2,springCloud]
---



# SpringSecurity整合Oauth2构建基本可用的服务(二)

[demo地址](https://github.com/AsummerCat/oauthdemo)

# 第一步 老规矩 导入pom

```java
     <dependency>
	    <groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-thymeleaf</artifactId>
		</dependency>   
```

```java
  <!--redis 相关配置信息   2.x版本之后默认使用lettuce -->
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-data-redis</artifactId>
		</dependency>
		<dependency>
      <groupId>io.lettuce</groupId>
      <artifactId>lettuce-core</artifactId>
		</dependency>   
		<dependency>
      <groupId>org.apache.commons</groupId>
      <artifactId>commons-pool2</artifactId>
      <version>2.6.2</version>
		</dependency>
```

```java
         <!--oauth2整合security的包-->
    <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-oauth2</artifactId>
		</dependency>
```

<!--more-->

# A服务器:创建认证授权服务器

- 继承 AuthorizationServerConfigurerAdapter 实现授权
- 开启@EnableAuthorizationServer  , @Configuration 注解
- 注入 RedisConnectionFactory 和 AuthenticationManager
- 实现三个方法

```java
声明单个客户端及其属性 最少一个 不然无法启动
 @Override
    public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
    }
    
   配置授权服务器终结点的非安全功能，如令牌存储、令牌自定义、用户批准和授予类型 请求方式
    @Override
    public void configure(AuthorizationServerEndpointsConfigurer endpoints) throws Exception {}
   
   配置（授权服务器安全配置器安全）
     @Override
    public void configure(AuthorizationServerSecurityConfigurer security) throws Exception {}
   
```

## 具体实现

```java
package com.linjingc.authentication.config;


import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.http.HttpMethod;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.oauth2.config.annotation.configurers.ClientDetailsServiceConfigurer;
import org.springframework.security.oauth2.config.annotation.web.configuration.AuthorizationServerConfigurerAdapter;
import org.springframework.security.oauth2.config.annotation.web.configuration.EnableAuthorizationServer;
import org.springframework.security.oauth2.config.annotation.web.configurers.AuthorizationServerEndpointsConfigurer;
import org.springframework.security.oauth2.config.annotation.web.configurers.AuthorizationServerSecurityConfigurer;
import org.springframework.security.oauth2.provider.token.store.redis.RedisTokenStore;

/**
 * 授权服务
 */
@Configuration
@EnableAuthorizationServer
public class AuthorizationServerConfiguration extends AuthorizationServerConfigurerAdapter {

    @Autowired
    @Qualifier("authenticationManagerBean")
    private AuthenticationManager authenticationManager;

    @Autowired
    RedisConnectionFactory redisConnectionFactory;


    /**
     * 声明单个客户端及其属性 最少一个 不然无法启动
     * @param clients
     * @throws Exception
     */
    @Override
    public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
        //这个地方后面会使用到 前缀表示密码加密类型
        String finalSecret = "{bcrypt}" + new BCryptPasswordEncoder().encode("123456");


//        clients
//                //内存模式
//                .inMemory()
//                .withClient("client_2")
//                .resourceIds(DEMO_RESOURCE_ID)
//                .authorizedGrantTypes("password", "refresh_token")
//                .scopes("select")
//                .authorities("oauth2")
//                .secret("123456")
//                .accessTokenValiditySeconds(100)
//                .refreshTokenValiditySeconds(100);

        //todo 查询数据库 或者查询内容只能选中一个
        //查询数据库
        clients.withClientDetails(new MyClientDetailsService());

    }

    /**
     * 配置授权服务器终结点的非安全功能，如令牌存储、令牌自定义、用户批准和授予类型 请求方式
     * @param endpoints
     * @throws Exception
     */
    @Override
    public void configure(AuthorizationServerEndpointsConfigurer endpoints) throws Exception {
        endpoints
                .tokenStore(new RedisTokenStore(redisConnectionFactory))
                .authenticationManager(authenticationManager)
                //允许 GET、POST 请求获取 token，即访问端点：oauth/token
                .allowedTokenEndpointRequestMethods(HttpMethod.GET, HttpMethod.POST);
    }


    /**
     * 配置（授权服务器安全配置器安全）
     * @param security
     * @throws Exception
     */
    @Override
    public void configure(AuthorizationServerSecurityConfigurer security) throws Exception {
        //允许表单认证
        security
                .tokenKeyAccess("permitAll()")
                .checkTokenAccess("permitAll()")
                .allowFormAuthenticationForClients();
        //自定义授权失败(forbidden)时返回信息
        security.accessDeniedHandler(new CustomAccessDeniedHandler());
    }


}

```

上面有几个自定义配置 下面描述

### * 首先 configure(ClientDetailsServiceConfigurer clients)

```java
   内存模式 和 数据库
   这里就是配置客户端的key 和权限的一些东西
   需要注意的是查询数据库 和内存模式 好像并不是共用的
   
    clients.withClientDetails(new MyClientDetailsService()); 
这里自定义一个进行查询 需要实现ClientDetailsService 中的loadClientByClientId方法
下面就贴出代码
```

#### 自定义的客户端类

```java
package com.linjingc.authentication.config;

import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.oauth2.provider.ClientDetails;

import java.util.Collection;
import java.util.Map;
import java.util.Set;

/**
 * 自定义客户端类
 */
public class MyClientDetails implements ClientDetails {
    private String clientId;
    private Set<String> resourceIds;
    private String clientSecret;
    private Set<String> scope;
    private Set<String> authorizedGrantTypes;
    private Collection<GrantedAuthority> authorities;
    private Integer accessTokenValiditySeconds;
    private Integer refreshTokenValiditySeconds;

    public MyClientDetails() {
    }

    public void setClientId(String clientId) {
        this.clientId = clientId;
    }

    public void setResourceIds(Set<String> resourceIds) {
        this.resourceIds = resourceIds;
    }

    public void setClientSecret(String clientSecret) {
        this.clientSecret = clientSecret;
    }

    public void setScope(Set<String> scope) {
        this.scope = scope;
    }

    public void setAuthorizedGrantTypes(Set<String> authorizedGrantTypes) {
        this.authorizedGrantTypes = authorizedGrantTypes;
    }

    public void setAuthorities(Collection<GrantedAuthority> authorities) {
        this.authorities = authorities;
    }

    public void setAccessTokenValiditySeconds(Integer accessTokenValiditySeconds) {
        this.accessTokenValiditySeconds = accessTokenValiditySeconds;
    }

    public void setRefreshTokenValiditySeconds(Integer refreshTokenValiditySeconds) {
        this.refreshTokenValiditySeconds = refreshTokenValiditySeconds;
    }

    @Override
    public String getClientId() {
        return clientId;
    }

    @Override
    public Set<String> getResourceIds() {
        return resourceIds;
    }

    @Override
    public boolean isSecretRequired() {
        return false;
    }

    @Override
    public String getClientSecret() {
        return clientSecret;
    }

    @Override
    public boolean isScoped() {
        return false;
    }

    @Override
    public Set<String> getScope() {
        return scope;
    }

    @Override
    public Set<String> getAuthorizedGrantTypes() {
        return authorizedGrantTypes;
    }

    @Override
    public Set<String> getRegisteredRedirectUri() {
        return null;
    }

    @Override
    public Collection<GrantedAuthority> getAuthorities() {
        return authorities;
    }

    @Override
    public Integer getAccessTokenValiditySeconds() {
        return accessTokenValiditySeconds;
    }

    @Override
    public Integer getRefreshTokenValiditySeconds() {
        return refreshTokenValiditySeconds;
    }

    @Override
    public boolean isAutoApprove(String scope) {
        return false;
    }

    @Override
    public Map<String, Object> getAdditionalInformation() {
        return null;
    }
}

```

#### 自定义获取客户端信息的信息类

```java
package com.linjingc.authentication.config;

import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.authority.SimpleGrantedAuthority;
import org.springframework.security.oauth2.provider.ClientDetails;
import org.springframework.security.oauth2.provider.ClientDetailsService;
import org.springframework.security.oauth2.provider.ClientRegistrationException;
import org.springframework.stereotype.Component;

import java.util.ArrayList;
import java.util.HashSet;
import java.util.List;
import java.util.Set;

/**
 * 获取客户端信息的信息类
 */
@Component
public class MyClientDetailsService implements ClientDetailsService {
    @Override
    public ClientDetails loadClientByClientId(String clientId) throws ClientRegistrationException {


        if (clientId.equals("client_1")) {
            MyClientDetails myClientDetails = new MyClientDetails();
            //客户端Id
            myClientDetails.setClientId("client_1");
            //resourceId 用于分配可以访问的资源服务
            Set<String> resourceIds = new HashSet<>();
            resourceIds.add("order");
            myClientDetails.setResourceIds(resourceIds);
            //认证类型
            Set<String> authorizedGrantTypes = new HashSet<>();
            authorizedGrantTypes.add("client_credentials");
            authorizedGrantTypes.add("refresh_token");
            myClientDetails.setAuthorizedGrantTypes(authorizedGrantTypes);

            //使用范围
            HashSet<String> scope = new HashSet<>();
            scope.add("select");
            scope.add("heihei");
            myClientDetails.setScope(scope);

            //权限信息
            List<GrantedAuthority> auths = new ArrayList<>();
            //这里添加权限 可以添加多个权限
            auths.add(new SimpleGrantedAuthority("add"));
            auths.add(new SimpleGrantedAuthority("update"));
            auths.add(new SimpleGrantedAuthority("delete"));
            auths.add(new SimpleGrantedAuthority("USER"));
            myClientDetails.setAuthorities(auths);

            //秘钥
            myClientDetails.setClientSecret("123456");

            //过期时间
            myClientDetails.setAccessTokenValiditySeconds(10000);
            myClientDetails.setRefreshTokenValiditySeconds(30000);
            return myClientDetails;
        }

        return null;
    }
}

```

这里可以用来查询数据库

### configure(AuthorizationServerSecurityConfigurer security)

```
这里配置了一个security.accessDeniedHandler(new CustomAccessDeniedHandler());
表示授权失败的时候自定义处理
```

#### 授权失败(forbidden)时返回信息

```java
package com.linjingc.authentication.config;

import com.linjingc.authentication.utils.HttpUtils;
import org.springframework.security.access.AccessDeniedException;
import org.springframework.security.web.access.AccessDeniedHandler;
import org.springframework.stereotype.Component;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

/**
 * 授权失败(forbidden)时返回信息
 */
@Component("customAccessDeniedHandler")
public class CustomAccessDeniedHandler implements AccessDeniedHandler {

    @Override
    public void handle(HttpServletRequest request, HttpServletResponse response, AccessDeniedException accessDeniedException) throws IOException, ServletException {
        // AJAX请求,使用response发送403
        if (HttpUtils.isAjaxRequest(request)) {
            response.sendError(403);
        } else if (!response.isCommitted()) {
            // 非AJAX请求，跳转403错误界面
            response.sendError(HttpServletResponse.SC_FORBIDDEN,
                    accessDeniedException.getMessage());
        }
    }
}

```

## 配置下WebSecurityConfigurerAdapter

```java
package com.linjingc.authentication.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.builders.WebSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.core.userdetails.User;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.crypto.password.NoOpPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.security.provisioning.InMemoryUserDetailsManager;

@Configuration
@EnableWebSecurity
public class SecurityConfiguration extends WebSecurityConfigurerAdapter {


    @Bean
    @Override
    protected UserDetailsService userDetailsService() {
        //BCryptPasswordEncoder bCryptPasswordEncoder = new BCryptPasswordEncoder();
//        password 方案一：明文存储，用于测试，不能用于生产
//        String finalPassword = "123456";
//        password 方案二：用 BCrypt 对密码编码
//        String finalPassword = bCryptPasswordEncoder.encode("123456");
        // password 方案三：支持多种编码，通过密码的前缀区分编码方式
        //String finalPassword = "{bcrypt}"+bCryptPasswordEncoder.encode("123456");
        InMemoryUserDetailsManager manager = new InMemoryUserDetailsManager();
        manager.createUser(User.withUsername("user_1").password("123456").authorities("USER").build());
        manager.createUser(User.withUsername("user_2").password("123456").authorities("USER").build());
        return manager;
    }

    /**
     * 这一步的配置是必不可少的，否则SpringBoot会自动配置一个AuthenticationManager,覆盖掉内存中的用户
     */
    @Bean
    @Override
    public AuthenticationManager authenticationManagerBean() throws Exception {
        return super.authenticationManagerBean();
    }

    // password 方案一：明文存储，用于测试，不能用于生产
    @Bean
    PasswordEncoder passwordEncoder() {
        return NoOpPasswordEncoder.getInstance();
    }

    // password 方案二：用 BCrypt 对密码编码
    //@Bean
    //PasswordEncoder passwordEncoder(){
    //    return new BCryptPasswordEncoder();
    //}

    // password 方案三：支持多种编码，通过密码的前缀区分编码方式,推荐
    //@Bean
    //PasswordEncoder passwordEncoder() {
    //    return PasswordEncoderFactories.createDelegatingPasswordEncoder();
    //}


    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                //开启路径不需要权限访问
                .antMatchers("/oauth/*", "/", "/error/*").permitAll()
                //其他路径都需要权限
                .anyRequest().authenticated();
    }


    /**
     * 放行静态资源
     */
    @Override
    public void configure(WebSecurity web) throws Exception {
        //解决静态资源被拦截的问题
        web.ignoring().antMatchers("/css/**");
        web.ignoring().antMatchers("/js/**");
        web.ignoring().antMatchers("/images/**");
        web.ignoring().antMatchers("/login/**");
        //解决服务注册url被拦截的问题
        web.ignoring().antMatchers("/resources/**");

    }
}
```

## 最后修改配置文件 添加redis

```java
  ## redis 配置
  redis:
    database: 4
    host: 112.74.43.131
    port: 6379
    password: jingbaobao

```

这样简单的认证服务器就有了

# B服务器:创建资源服务器

- 继承 ResourceServerConfigurerAdapter 实现资源服务器

- 开启@EnableResourceServer, @Configuration 注解

- 注入 RedisConnectionFactory

- 导入pom文件 跟上面的内容一样 

- 实现三个方法

- 这边redis配置跟上面一样 就不写了

  这边跟上面差不多就直接贴代码了

## 具体代码

```java
package com.linjingc.resources.config;

import com.linjingc.resources.config.errormsg.AuthExceptionEntryPoint;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.http.SessionCreationPolicy;
import org.springframework.security.oauth2.config.annotation.web.configuration.EnableResourceServer;
import org.springframework.security.oauth2.config.annotation.web.configuration.ResourceServerConfigurerAdapter;
import org.springframework.security.oauth2.config.annotation.web.configurers.ResourceServerSecurityConfigurer;
import org.springframework.security.oauth2.provider.token.store.redis.RedisTokenStore;


/**
 * 配置资源服务器
 */
@Configuration
@EnableResourceServer
public class ResourceServerConfiguration extends ResourceServerConfigurerAdapter {
    private static final String DEMO_RESOURCE_ID = "order";

    @Autowired
    RedisConnectionFactory redisConnectionFactory;

    //内部关联了ResourceServerSecurityConfigurer和HttpSecurity。前者与资源安全配置相关，后者与http安全配置相关
    @Override
    public void configure(ResourceServerSecurityConfigurer resources) throws Exception {
        //resourceId 用于分配给可授予的clientId
        //stateless  标记以指示在这些资源上仅允许基于令牌的身份验证
        //tokenStore token的存储方式
        resources.resourceId(DEMO_RESOURCE_ID).stateless(true).tokenStore(new RedisTokenStore(redisConnectionFactory));

        //认证异常流程处理返回
        resources.authenticationEntryPoint(new AuthExceptionEntryPoint());
        // .accessDeniedHandler(CustomAccessDeniedHandler);
    }

    @Override
    public void configure(HttpSecurity http) throws Exception {
        http.sessionManagement().sessionCreationPolicy(SessionCreationPolicy.IF_REQUIRED);
        http.authorizeRequests()
                //开启路径不需要权限访问
                .antMatchers("/").permitAll()
                //其他路径都需要权限
                .antMatchers("/product/**").access("#oauth2.hasScope('select') and hasPermission('delete')")
                //配置order访问控制，必须认证过后才可以访问
                .antMatchers("/order/**").authenticated()
                .anyRequest().authenticated();
    }
}

```



### 上面的      resources.authenticationEntryPoint(new AuthExceptionEntryPoint());

```java
package com.linjingc.resources.config.errormsg;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.security.core.AuthenticationException;
import org.springframework.security.web.AuthenticationEntryPoint;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

/**
 * 自定义AuthExceptionEntryPoint
 * 用于token校验失败返回信息
 */
public class AuthExceptionEntryPoint implements AuthenticationEntryPoint {

    @Value("${basic.errorUrl.unauthorized}")
    private String url;


    @Override
    public void commence(HttpServletRequest request, HttpServletResponse response,
                         AuthenticationException authException)
            throws ServletException {

        try {
            response.sendRedirect(url);
        } catch (Exception e) {
            throw new ServletException();
        }
    }
}
```

可以自定义跳转的错误页面 这边配置在认证服务器上的url  上面会有个自定义错误页面

## 最后配置文件

```java
spring:
  application:
    name: resouricesOauth
  ## redis 配置
  redis:
    database: 4
    host: 112.74.43.131
    port: 6379
    password: jingbaobao

server:
  port: 8200

logging.level.org.springframework.security: DEBUG

security:
  oauth2:
 #   resource:
#      token-info-uri: http://localhost:9005/oauth/check_token
#      user-info-uri: http://localhost:9005/user
   # authorization:
#      check-token-access: http://localhost:9005/oauth/check_token
  #  sso:
      login-path: /


basic:
  errorUrl:
    unauthorized: http://localhost:8100/error/401


```



这样就搭建起来可用了

# 下面这个是完整的登录服务+资源访问 

## 导入pom文件 这边需要在服务端进行http请求

```java
  <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-thymeleaf</artifactId>
        </dependency>
        

<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-configuration-processor</artifactId>
            <optional>true</optional>
        </dependency>

        <!-- oauth2请求工具类-->
        <dependency>
            <groupId>org.apache.oltu.oauth2</groupId>
            <artifactId>org.apache.oltu.oauth2.client</artifactId>
            <version>1.0.2</version>
        </dependency>

```



## 创建登录入口

```java
  @RequestMapping(value = "/doLogin", method = RequestMethod.POST)
    public String doLogin(String username, String password, HttpServletRequest request, HttpServletResponse response) throws IOException {
        String apiToken;
        try {
            apiToken = oAuthClientUtil.getApiToken(username, password);
        } catch (OAuthProblemException e) {
            //重置response
            response.reset();
            //设置编码格式
            response.setCharacterEncoding("UTF-8");
            response.setContentType("application/json;charset=UTF-8");
            PrintWriter pw = response.getWriter();
            pw.write(e.getDescription());
            pw.flush();
            pw.close();
            return "";
        }
        request.getSession(false).setAttribute("token",apiToken);
    return  "hello";
    }

```

### oauth工具类

```java
package com.linjingc.client.utils;

import lombok.Data;
import org.apache.oltu.oauth2.client.OAuthClient;
import org.apache.oltu.oauth2.client.URLConnectionClient;
import org.apache.oltu.oauth2.client.request.OAuthClientRequest;
import org.apache.oltu.oauth2.client.response.OAuthAccessTokenResponse;
import org.apache.oltu.oauth2.common.OAuth;
import org.apache.oltu.oauth2.common.exception.OAuthProblemException;
import org.apache.oltu.oauth2.common.exception.OAuthSystemException;
import org.apache.oltu.oauth2.common.message.types.GrantType;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;

/**
 * 请求oAuth授权工具类
 *
 * @author cxc
 */
@Component
@Data
@ConfigurationProperties(prefix = "basic.oauth2")
public class OAuthClientUtil {
    private String authorizationUrl;
    private String clientId;
    private String secret;
    private String scope;


    private static Logger logger = LoggerFactory.getLogger(OAuthClientUtil.class.getName());

    /**
     * 获取token
     *
     * @param username
     * @param password
     * @return
     * @throws OAuthProblemException
     */
    public String getApiToken(String username, String password) throws OAuthProblemException {
        logger.info("api getApiToken");
        String accessToken = null;
        OAuthClient oAuthClient = new OAuthClient(new URLConnectionClient());
        try {
            OAuthClientRequest request = OAuthClientRequest
                    .tokenLocation(authorizationUrl)
                    .setGrantType(GrantType.PASSWORD)
                    .setUsername(username)
                    .setPassword(password)
                    .setClientId(clientId)
                    .setClientSecret(secret)
                    .setScope(scope)
                    .buildQueryMessage();

            request.addHeader("Accept", "application/json");
            request.addHeader("Content-Type", "application/json");

            OAuthAccessTokenResponse oAuthResponse = oAuthClient.accessToken(request, OAuth.HttpMethod.POST); //去服务端请求access_token，并返回响应
            accessToken = oAuthResponse.getAccessToken(); //获取服务端返回过来的access_token
            logger.info("api token: " + accessToken);
        } catch (OAuthSystemException e) {
            e.printStackTrace();
        }
        return accessToken;
    }

    /**
     * 刷新token
     *
     * @param token
     * @return
     * @throws OAuthProblemException
     */
    public String refreshToken(String token) throws OAuthProblemException {
        String accessToken = null;

        OAuthClient oAuthClient = new OAuthClient(new URLConnectionClient());
        try {
            OAuthClientRequest request = OAuthClientRequest
                    .tokenLocation(authorizationUrl)
                    .setGrantType(GrantType.REFRESH_TOKEN)
                    .setClientId(clientId)
                    .setClientSecret(secret)
                    .setScope(scope)
                    .setRefreshToken(token)
                    .buildQueryMessage();

            request.addHeader("Accept", "application/json");
            request.addHeader("Content-Type", "application/json");

            OAuthAccessTokenResponse oAuthResponse = oAuthClient.accessToken(request, OAuth.HttpMethod.POST); //去服务端请求access_token，并返回响应
            accessToken = oAuthResponse.getAccessToken(); //获取服务端返回过来的access_token
            logger.info("api token: " + accessToken);
        } catch (OAuthSystemException e) {
            e.printStackTrace();
        }
        return accessToken;
    }

}

```

## 资源服务器的配置

```java
package com.linjingc.client.config;

import com.linjingc.client.config.errormsg.AuthExceptionEntryPoint;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.http.SessionCreationPolicy;
import org.springframework.security.oauth2.config.annotation.web.configuration.EnableResourceServer;
import org.springframework.security.oauth2.config.annotation.web.configuration.ResourceServerConfigurerAdapter;
import org.springframework.security.oauth2.config.annotation.web.configurers.ResourceServerSecurityConfigurer;
import org.springframework.security.oauth2.provider.token.store.redis.RedisTokenStore;


/**
 * 配置资源服务器
 */
@Configuration
@EnableResourceServer
public class ResourceServerConfiguration extends ResourceServerConfigurerAdapter {
    private static final String DEMO_RESOURCE_ID = "order";

    @Autowired
    RedisConnectionFactory redisConnectionFactory;

    //内部关联了ResourceServerSecurityConfigurer和HttpSecurity。前者与资源安全配置相关，后者与http安全配置相关
    @Override
    public void configure(ResourceServerSecurityConfigurer resources) throws Exception {
        //resourceId 用于分配给可授予的clientId
        //stateless  标记以指示在这些资源上仅允许基于令牌的身份验证
        //tokenStore token的存储方式
        resources.resourceId(DEMO_RESOURCE_ID).stateless(true).tokenStore(new RedisTokenStore(redisConnectionFactory));

        //认证异常流程处理返回
        resources.authenticationEntryPoint(new AuthExceptionEntryPoint());
    }

    @Override
    public void configure(HttpSecurity http) throws Exception {
        http.sessionManagement().sessionCreationPolicy(SessionCreationPolicy.IF_REQUIRED);
        http.authorizeRequests()
                //开启路径不需要权限访问
                .antMatchers("/", "/login", "/doLogin").permitAll()
                //其他路径都需要权限
                .anyRequest().authenticated();
    }
}


```

## 认证异常流程处理返回

```java
package com.linjingc.client.config.errormsg;

import com.linjingc.client.utils.OAuthClientUtil;
import org.apache.oltu.oauth2.common.exception.OAuthProblemException;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.autoconfigure.security.oauth2.client.OAuth2ClientProperties;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.security.core.AuthenticationException;
import org.springframework.security.oauth2.provider.error.DefaultWebResponseExceptionTranslator;
import org.springframework.security.oauth2.provider.error.OAuth2AuthenticationEntryPoint;
import org.springframework.security.oauth2.provider.error.WebResponseExceptionTranslator;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

/**
 * 自定义AuthExceptionEntryPoint
 * 用于token校验失败返回信息
 */
public class AuthExceptionEntryPoint extends OAuth2AuthenticationEntryPoint {

    @Value("${basic.errorUrl.unauthorized}")
    private String url;
    @Autowired
    private OAuth2ClientProperties oAuth2ClientProperties;
    private WebResponseExceptionTranslator<?> exceptionTranslator = new DefaultWebResponseExceptionTranslator();
    @Autowired
    private OAuthClientUtil oAuthClientUtil;


    @Override
    public void commence(HttpServletRequest request, HttpServletResponse response,
                         AuthenticationException authException)
            throws ServletException {
        try {

            //解析异常，如果是401则处理
            ResponseEntity<?> result = exceptionTranslator.translate(authException);
            if (result.getStatusCode() == HttpStatus.UNAUTHORIZED) {
                try {
                    String token = oAuthClientUtil.refreshToken(request.getSession().getAttribute("token").toString());
                    request.getSession(false).setAttribute("token", token);
                    request.getRequestDispatcher(request.getRequestURI()).forward(request, response);

                } catch (OAuthProblemException e) {
                    System.out.println(e.toString());
                    response.sendRedirect(url);
                }

            } else {
                //如果不是401异常，则以默认的方法继续处理其他异常
                super.commence(request, response, authException);
            }

        } catch (Exception e) {
            throw new ServletException();
        }
    }
}

```

## 配置文件

```java
spring:
  application:
    name: resouricesOauth
  ## redis 配置
  redis:
    database: 4
    host: 112.74.43.136
    port: 6379
    password: jingbaobao

server:
  port: 8300


basic:
  errorUrl:
    unauthorized: http://localhost:8100/error/401
  oauth2:
    authorizationUrl: http://localhost:8100/oauth/token
    clientId: client_2
    secret: 123456
    scope: select

```

## html页面

```java
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml"
      xmlns:th="http://www.thymeleaf.org" xmlns:sec="http://www.thymeleaf.org/thymeleaf-extras-springsecurity3">
<head>
    <title>Spring Security Example </title>
</head>
<body>
<form th:action="@{/doLogin}" method="post">
    <div><label> 用户名 : <input type="text" name="username"/> </label></div>
    <div><label> 密  码 : <input type="password" name="password"/> </label></div>
    <div><input type="submit" value="登录"/></div>
</form>
</body>
</html>

```



# END