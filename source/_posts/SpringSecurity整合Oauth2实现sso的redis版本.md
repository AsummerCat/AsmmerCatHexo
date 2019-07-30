---
title: SpringSecurity整合Oauth2实现sso的redis版本
date: 2019-07-30 23:53:19
tags: [springCloud,Oauth2,Security,单点登录]
---

# 基于Oauth2实现SSO整合Redis

[demo地址](https://github.com/AsummerCat/oauth-sso-demo)

# 注意事项

## 开启注解

可以使用 `@PreAuthorize("hasAuthority('user:update')")`

```
@EnableGlobalMethodSecurity(prePostEnabled = true)
```

<!--more-->

# 构建认证服务器

## 老规矩导入pom

```
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-oauth2</artifactId>
        </dependency>
        
        <!-- 导入Spring session和redis整合的坐标 如果只要redis的话就换单独redis的坐标-->
         <dependency>
            <groupId>org.springframework.data</groupId>
            <artifactId>spring-data-redis</artifactId>
        </dependency>
        <dependency>
            <groupId>io.lettuce</groupId>
            <artifactId>lettuce-core</artifactId>
        </dependency>
        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-pool2</artifactId>
            <version>${commons-pool2.version}</version>
        </dependency>
```

## 创建SecurityConfiguration配置类

控制权限登录信息之类的

```java
/**
 * Security配置类
 *
 * @author cxc
 * @date 2019年7月11日16:38:56
 */
@Configuration
@EnableWebSecurity
public class SecurityConfiguration extends WebSecurityConfigurerAdapter {

    @Bean
    @Override
    protected UserDetailsService userDetailsService() {
        BCryptPasswordEncoder bCryptPasswordEncoder = new BCryptPasswordEncoder();
        String finalPassword = "{bcrypt}" + bCryptPasswordEncoder.encode("123456");

        InMemoryUserDetailsManager manager = new InMemoryUserDetailsManager();

        manager.createUser(User.withUsername("583188551").password(finalPassword).authorities("ROLE_USER").build());
        manager.createUser(User.withUsername("724307597").password(finalPassword).authorities("ROLE_USER").build());
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


    /**
     * 支持多种编码，通过密码的前缀区分编码方式,推荐
     *
     * @return
     */
    @Bean
    PasswordEncoder passwordEncoder() {
        return PasswordEncoderFactories.createDelegatingPasswordEncoder();
    }


    @Override
    protected void configure(HttpSecurity http) throws Exception {

        http.formLogin().loginPage("/authentication/require")
                .loginProcessingUrl("/authentication/form")
                .and().authorizeRequests()
                //不拦截
                .antMatchers("/authentication/require", "/authentication/form","/oauth/**").permitAll()
                //其他请求全部需要授权
                .anyRequest().authenticated()
                .and()
                //关闭跨域保护
                .csrf().disable();


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
        //解决服务注册url被拦截的问题
        web.ignoring().antMatchers("/resources/**");
    }
}
```

这边没啥注意事项

## 创建AuthorizationServerConfiguration 授权服务类

需要实现`AuthorizationServerConfigurerAdapter`

开启两个注解`@Configuration` `@EnableAuthorizationServer`



```java

/**
 * 授权服务
 *
 * @author cxc
 * @date 2019年7月11日16:38:56
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
     *
     * @param clients
     * @throws Exception
     */
    @Override
    public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
        //这个地方后面会使用到 前缀表示密码加密类型
        String finalSecret = "{bcrypt}" + new BCryptPasswordEncoder().encode("123456");

        clients.inMemory()
                .withClient("client")
                .secret(finalSecret)
                .authorizedGrantTypes("authorization_code", "refresh_token")
                .scopes("all")
//                .accessTokenValiditySeconds(3600)
                .accessTokenValiditySeconds(30)
                .refreshTokenValiditySeconds(1000)
                .redirectUris("http://my.cloud.com/login")
                .autoApprove(true);
    }


    @Bean
    public TokenStore tokenStore() {
        return new RedisTokenStore(redisConnectionFactory);
    }

    /**
     * 配置授权服务器终结点的非安全功能，如令牌存储、令牌自定义、用户批准和授予类型 请求方式
     *
     * @param endpoints
     * @throws Exception
     */
    @Override
    public void configure(AuthorizationServerEndpointsConfigurer endpoints) throws Exception {
        endpoints
                .tokenStore(tokenStore())
                .authenticationManager(authenticationManager)
                //允许 GET、POST 请求获取 token，即访问端点：oauth/token
                .allowedTokenEndpointRequestMethods(HttpMethod.GET, HttpMethod.POST)
                //刷新token
                .reuseRefreshTokens(true);
    }

    /**
     * 配置（授权服务器安全配置器安全）
     *
     * @param security
     * @throws Exception
     */
    @Override
    public void configure(AuthorizationServerSecurityConfigurer security) throws Exception {
        //允许表单认证
        security.allowFormAuthenticationForClients().tokenKeyAccess("permitAll()")//公开/oauth/token的接口
                .checkTokenAccess("permitAll()");
               // .allowFormAuthenticationForClients(); //允许表单认证  这段代码在授权码模式下会导致无法根据code　获取token　

    }
}
```

这边认证成功 返回的token是一串UUID 如果你需要修改为jwt模式的话

多添加一些内容

```java
configure(AuthorizationServerEndpointsConfigurer endpoints)方法中加入
.accessTokenConverter(jwtAccessTokenConverter())


--->
endpoints.tokenStore(tokenStore()).accessTokenConverter(jwtAccessTokenConverter())
--->

/**
     * 生成JTW token
     *
     * @return
     */
    @Bean
    public JwtAccessTokenConverter jwtAccessTokenConverter() {
        JwtAccessTokenConverter converter = new JwtAccessTokenConverter();
        return converter;
    }
```



注意点:

```java
.tokenKeyAccess("permitAll()")//公开/oauth/token的接口
                .checkTokenAccess("permitAll()");  //检测是否认证
```

# 自定义登录入口

```java
@Controller
public class LoginController {

    @RequestMapping("/")
    @ResponseBody
    public String index() {
        return "进入认证服务器";
    }

    @GetMapping("/authentication/require")
    public String login() {
        return "login";
    }
}
```

### 登录页

```java
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
<form th:action="@{/authentication/form}" method="post">
    <div><label> 用户名 : <input type="text" name="username"/> </label></div>
    <div><label> 密  码 : <input type="password" name="password"/> </label></div>
    <div><input type="submit" value="登录"/></div>
</form>
</body>
</html>
```

这样认证服务器基本构建完成

# 构建单点登录的SSO服务器

## 这边需要注意的是如果实现了刷新token的模式的话

sso服务器session过期时间设置需要跟 授权服务器设置的token有效时间一样 不然可能产生sso服务器这边不会刷新token退出导致 无法访问资源服务器 报token无效 因为授权服务器那边已经过期抹除了 sso这边还没刷新

sso这边的seesion超时时间可以在配置文件中添加

```java
server:
  servlet:
    session:
      timeout: 1m
```

## 这边的pom只要导入 oauth2的就可以了

```java
<dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-oauth2</artifactId>
        </dependency>
```

## 创建SecurityConfiguration配置类

注意需要添加`@EnableOAuth2Sso`注解

表示这是一个sso的客户端 

```java

@Configuration
@EnableOAuth2Sso
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    //@Autowired
    //private MyAuthenticationSuccessHandler myAuthenticationSuccessHandler;
    @Override
    protected void configure(HttpSecurity http) throws Exception {


        http.csrf().disable();
        http.authorizeRequests()
                //不需要权限访问
                .antMatchers("/**.html", "/**.html", "/**.css", "/img/**", "/**.js", "/").permitAll()
                .anyRequest().authenticated();
}
```

注意点: 

* 这边有一个问题sso登录成功后的如果需要自定义授权成功后置处理器的话使用

* ```java
  .and().formLogin().successHandler(new MyAuthenticationSuccessHandler());
  ```

  这种方式是无法注入的

  解决办法 : 直接使用Spring 重新手动赋值给OAuth2ClientAuthenticationProcessingFilter的setAuthenticationSuccessHandler方法

  ```java
  
  
  /**
   * 手动注入spring
   * setAuthenticationSuccessHandler
   *
   * @author cxc
   */
  @Component
  public class DefaultRolesPrefixPostProcessor implements BeanPostProcessor {
  
      @Override
      public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
          if (bean instanceof FilterChainProxy) {
  
              FilterChainProxy chains = (FilterChainProxy) bean;
  
              for (SecurityFilterChain chain : chains.getFilterChains()) {
                  for (Filter filter : chain.getFilters()) {
                      if (filter instanceof OAuth2ClientAuthenticationProcessingFilter) {
                          OAuth2ClientAuthenticationProcessingFilter oAuth2ClientAuthenticationProcessingFilter = (OAuth2ClientAuthenticationProcessingFilter) filter;
  //手动注入
                        oAuth2ClientAuthenticationProcessingFilter.setAuthenticationSuccessHandler(new MyAuthenticationSuccessHandler());
                      }
                  }
              }
          }
          return bean;
      }
  }
  ```

* 自定义的登录成功后置处理器

```java

/**
 * 自定义登录成功 后置处理
 *
 * @author cxc
 * @date 2019/7/2 21:17
 */
@Slf4j
public class MyAuthenticationSuccessHandler extends SavedRequestAwareAuthenticationSuccessHandler {



    @Override
    public void onAuthenticationSuccess(HttpServletRequest request,
                                        HttpServletResponse response, Authentication authentication)
            throws ServletException, IOException {
        //这里就可以写自定义的方法 比如创建日志
        System.out.println("创建日志");
        OAuth2AuthenticationDetails details = (OAuth2AuthenticationDetails) authentication.getDetails();
        super.onAuthenticationSuccess(request, response, authentication);
    }
}
```

这样服务基本搭建完成了

然后就是添加配置文件信息 实现sso的内容了

```java
auth-server: http://my.oauth.com:8200

security:
  oauth2:
    client:
      client-id: client
      client-secret: 123456
        # 获取token地址
      access-token-uri: ${auth-server}/oauth/token
      # 授权地址
      user-authorization-uri: ${auth-server}/oauth/authorize
    resource:
      token-info-uri: ${auth-server}/oauth/check_token
```



# 构建资源服务器



资源服务器这边 就更简单了 直接开启`@EnableResourceServer`就可以了

```java

@EnableResourceServer
public abstract class ResServerConfig extends ResourceServerConfigurerAdapter {


    @Override
    public void configure(ResourceServerSecurityConfigurer resources) throws Exception {
        super.configure(resources);

//        resources.authenticationEntryPoint(new LLGAuthenticationEntryPoint());

    }
}

```



## 配置文件部分

```java
auth-server: http://my.oauth.com:8200

security:
  oauth2:
    client:
      client-id: client
      client-secret: 123456
      access-token-uri: ${auth-server}/oauth/token
      user-authorization-uri: ${auth-server}/oauth/authorize
    resource:
      token-info-uri: ${auth-server}/oauth/check_token
```



这样的话 就可以实现资源服务器了

配置文件

# 客户端退出 实现单点退出

这边的话 如果因为是sso客户端 获取seesion中的token 去访问资源的

如果客户端这边没有退出的话 可以直接抓取到token 然后访问资源

或者说退出了客户端 没有通知授权服务器的话  基本就相当于客户端游离了 但是token和刷新token依然 有效 带个access_toekn就可以访问资源了

所有客户端退出 也要删除授权服务器上的token相关信息

这边可以这么实现

## 首先是客户端实现部分 SecurityConfig修改配置类

```
@Configuration
@EnableOAuth2Sso
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Autowired
    MySsoLogoutHandler mySsoLogoutHandler;

    /**
     * 认证服务器退出地址
     */
    @Value("${auth-server}/${auth-server-logout-method}")
    String logoutUrl;

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.csrf().disable();
        http.authorizeRequests()
                //不需要权限访问
                .antMatchers("/**.html", "/**.html", "/**.css", "/img/**", "/**.js", "/").permitAll()
                .anyRequest().authenticated().
                // this LogoutHandler invalidate user token from SSO
                        and().logout().addLogoutHandler(mySsoLogoutHandler).logoutSuccessUrl(logoutUrl);
                //.deleteCookies("JSESSIONID", "ANY_OTHER_COOKIE").permitAll().and().csrf().csrfTokenRepository(CookieCsrfTokenRepository.withHttpOnlyFalse());

    }
}
```

这边主要操作就是客户端退出后再发送请求通知授权服务器退出

这边如果需要删除token信息的话 你可以再配置一个SecurityContextLogoutHandler

进行手动remove token

```java


/**
 * @deprecated 自定义oauth2 注销方法
 */

@Component
@Qualifier("mySsoLogoutHandler")
public class MySsoLogoutHandler extends SecurityContextLogoutHandler implements LogoutHandler {

    @Value("${auth-server}/${auth-server-logout-method}")
    String logoutUrl;

    @Override
    public void logout(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Authentication authentication) {
        Object details = authentication.getDetails();
        if (details.getClass().isAssignableFrom(OAuth2AuthenticationDetails.class)) {

            String accessToken = ((OAuth2AuthenticationDetails) details).getTokenValue();

            RestTemplate restTemplate = new RestTemplate();

            MultiValueMap<String, String> params = new LinkedMultiValueMap<>();
            params.add("access_token", accessToken);

            HttpHeaders headers = new HttpHeaders();
            headers.add("Authorization", "bearer " + accessToken);

            HttpEntity<String> request = new HttpEntity(params, headers);

            HttpMessageConverter formHttpMessageConverter = new FormHttpMessageConverter();
            HttpMessageConverter stringHttpMessageConverternew = new StringHttpMessageConverter();
            restTemplate.setMessageConverters(Arrays.asList(new HttpMessageConverter[]{formHttpMessageConverter, stringHttpMessageConverternew}));
            try {
                ResponseEntity<String> response = restTemplate.exchange(logoutUrl, HttpMethod.POST, request, String.class);
            } catch (HttpClientErrorException e) {
                //LOGGER.error("HttpClientErrorException invalidating token with SSO authorization server. response.status code: {}, server URL: {}", e.getStatusCode(), logoutUrl);
            }

        }
    }
}
```

## 授权服务器这边的话

就设置一个`/oauth/exit`端点

```java
   /**
     * 退出操作
     */
    @RequestMapping("oauth/exit")
    public void exit(HttpServletRequest request, HttpServletResponse response) {
        //首先移除认证服务器上的session
        new SecurityContextLogoutHandler().logout(request, null, null);
        try {
            //如果存在token 移除token 如果没有token表示successUrl转发
            String token = request.getHeader("Authorization");
            if (token != null) {
                String tokenValue = token.replace("bearer ", "").trim();
                OAuth2AccessToken oAuth2AccessToken = tokenStore.readAccessToken(tokenValue);
                if (oAuth2AccessToken != null) {
                    //移除access_token
                    tokenStore.removeAccessToken(oAuth2AccessToken);
                    //移除refresh_token
                    tokenStore.removeRefreshToken(oAuth2AccessToken.getRefreshToken());
                }
            }else{
                //sending back to client app
                response.sendRedirect(request.getHeader("referer"));
            }
            System.out.println("退出授权服务器");
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
```

