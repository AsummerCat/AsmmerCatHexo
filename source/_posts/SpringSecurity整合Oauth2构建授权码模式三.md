---
title: SpringSecurity整合Oauth2构建授权码模式三
date: 2019-07-11 22:24:29
tags: [Security,springCloud,Oauth2]
---

# SpringSecurity整合Oauth2构建授权码模式三

[demo地址](https://github.com/AsummerCat/oauthdemo) 里面的

 [authentication-qq](https://github.com/AsummerCat/oauthdemo/tree/master/authentication-qq)

[client-qq](https://github.com/AsummerCat/oauthdemo/tree/master/client-qq)

# 首先还是导入pom

```java
这不就写了
```

# 授权流程

```
例如 第三方登录:
   浏览器              客户端             认证端          资源端
  请求客户端微信登录  -> 发送请求给认证端带上客户端信息及其回调地址-> 认证端要求用户登录->登录后授权后获取code ->再根据回调地址返回code-> 客户端接收code 进行请求认证端 获取token->
  获取token后请求资源端获取用户信息->客户端进行数据库匹配用户信息 匹配成功后 关闭页面 返回session 登录成功;
```

<!--more-->

# 认证端代码



需要注意的是 因为是第三方授权 所有客户密码信息并不需要扭转到客户端 

认证端 认证的前提下是用户登录了

# SecurityConfiguration Security通用配置

这里需要把授权的地址拦截下 要登录后才能使用

```
package com.linjingc.authenticationqq.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.core.userdetails.User;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.crypto.password.NoOpPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.security.provisioning.InMemoryUserDetailsManager;
import org.springframework.security.web.util.matcher.AntPathRequestMatcher;

@Configuration
@EnableWebSecurity
public class SecurityConfiguration extends WebSecurityConfigurerAdapter {

    @Bean
    @Override
    protected UserDetailsService userDetailsService() {
        InMemoryUserDetailsManager manager = new InMemoryUserDetailsManager();
        // 创建两个 qq 用户
        manager.createUser(User.withUsername("583188551").password("123456").authorities("USER").build());
        manager.createUser(User.withUsername("724307597").password("123456").authorities("USER").build());
        return manager;
    }


    @Override
    @Bean
    public AuthenticationManager authenticationManagerBean() throws Exception {
        return super.authenticationManagerBean();
    }

    @Bean
    PasswordEncoder passwordEncoder() {
        return NoOpPasswordEncoder.getInstance();
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.
                requestMatchers()
                // /oauth/authorize link org.springframework.security.oauth2.provider.endpoint.AuthorizationEndpoint
                // 必须登录过的用户才可以进行 oauth2 的授权码申请
                .antMatchers("/", "/home", "/login", "/oauth/authorize")
                .and()
                .authorizeRequests()
                .anyRequest().permitAll()
                .and()
                .formLogin()
                .loginPage("/login")
                .and()
                .httpBasic()
                .disable()
                .exceptionHandling()
                .accessDeniedPage("/login?authorization_error=true")
                .and()
                // TODO: put CSRF protection back into this endpoint
                .csrf()
                .requireCsrfProtectionMatcher(new AntPathRequestMatcher("/oauth/authorize"))
                .disable();
    }
}

```

# 服务端配置

```java
/**
 * 认证服务 和 资源服务
 *
 * @author cxc
 */
@Configuration
public class OAuth2ServerConfig {

    private static final String QQ_RESOURCE_ID = "qq";


    /**
     * 认证服务
     */
    @Configuration
    @EnableAuthorizationServer
    protected static class AuthorizationServerConfiguration extends AuthorizationServerConfigurerAdapter {

        @Autowired
        @Qualifier("authenticationManagerBean")
        private AuthenticationManager authenticationManager;

        @Override
        public void configure(ClientDetailsServiceConfigurer clients) throws Exception {

            clients.inMemory().withClient("aiqiyi")
                    .resourceIds(QQ_RESOURCE_ID)
                    .authorizedGrantTypes("authorization_code", "refresh_token", "implicit")
                    .authorities("ROLE_CLIENT")
                    .scopes("get_user_info", "get_fanslist")
                    .secret("secret")
                    .redirectUris("http://localhost:8400/aiqiyi/qq/redirect")
                    //登录后绕过批准询问(/oauth/confirm_access)
                    .autoApprove(true)
                    .autoApprove("get_user_info");
        }

        @Bean
        public ApprovalStore approvalStore() {
            TokenApprovalStore store = new TokenApprovalStore();
            store.setTokenStore(tokenStore());
            return store;
        }

        @Autowired
        RedisConnectionFactory redisConnectionFactory;

        @Bean
        public TokenStore tokenStore() {
            return new RedisTokenStore(redisConnectionFactory);
        }

        @Override
        public void configure(AuthorizationServerEndpointsConfigurer endpoints) {
            endpoints.tokenStore(tokenStore())
                    .authenticationManager(authenticationManager)
                    .allowedTokenEndpointRequestMethods(HttpMethod.GET, HttpMethod.POST);
        }

        @Override
        public void configure(AuthorizationServerSecurityConfigurer oauthServer) {
            //允许表单认证
            oauthServer.realm(QQ_RESOURCE_ID).allowFormAuthenticationForClients();
        }

    }

    /**
     * 资源服务
     */
    @Configuration
    @EnableResourceServer()
    protected static class ResourceServerConfiguration extends ResourceServerConfigurerAdapter {

        @Override
        public void configure(ResourceServerSecurityConfigurer resources) {
            resources.resourceId(QQ_RESOURCE_ID).stateless(true);
        }


        @Override
        public void configure(HttpSecurity http) throws Exception {
            http.requestMatchers()
                    // 保险起见，防止被主过滤器链路拦截
                    .antMatchers("/qq/**").and()
                    .authorizeRequests().anyRequest().authenticated()
                    .and()
                    .authorizeRequests()
                    .antMatchers("/qq/info/**").access("#oauth2.hasScope('get_user_info')")
                    .antMatchers("/qq/fans/**").access("#oauth2.hasScope('get_fanslist')");
        }
    }


}

```

# 客户端 

```java
@RestController
@Slf4j
public class QQCallbackController {

//    withClient("aiqiyi")
//    .authorizedGrantTypes("authorization_code","refresh_token", "implicit")
//    .authorities("ROLE_CLIENT")
//    .scopes("get_user_info","get_fanslist")
//    .secret("secret")
//    .redirectUris("http://localhost:8081/aiqiyi/qq/redirect")
//    .autoApprove(true)
//    .autoApprove("get_user_info")

    @Autowired
    RestTemplate restTemplate;

    @RequestMapping("/aiqiyi/qq/redirect")
    public String getToken(@RequestParam String code) {
        log.info("receive code {}", code);
        HttpHeaders headers = new HttpHeaders();
        headers.setContentType(MediaType.APPLICATION_FORM_URLENCODED);
        MultiValueMap<String, String> params = new LinkedMultiValueMap<>();
        params.add("grant_type", "authorization_code");
        params.add("code", code);
        params.add("client_id", "aiqiyi");
        params.add("client_secret", "secret");
        params.add("redirect_uri", "http://localhost:8400/aiqiyi/qq/redirect");
        HttpEntity<MultiValueMap<String, String>> requestEntity = new HttpEntity<>(params, headers);
        ResponseEntity<String> response = restTemplate.postForEntity("http://localhost:8500/oauth/token", requestEntity, String.class);
        String token = response.getBody();
        log.info("token => {}", token);
        return token;
    }

}

```

这边写入一个回调地址 需要注意的是 这里的回调地址 需要跟你申请在服务端上的回调地址一样 不然会返回 回调地址不正确的错误

然后发起一个请求 根据code 获取token 再根据这个token 获取相应的服务器资源

获取完毕后 该登录就登录 该失败就失败;