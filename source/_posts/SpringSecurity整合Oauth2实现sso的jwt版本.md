---
title: SpringSecurity整合Oauth2实现sso的jwt版本
date: 2019-07-31 19:47:59
tags: [SpringCloud,Oauth2,Security,单点登录]
---

# 基于Oauth2实现SSO整合Redis

[demo地址](https://github.com/AsummerCat/oauth-sso-jwt-demo)

# 导入pom.xml

这边只要导入

```
  <dependency>
            <groupId>org.springframework.security</groupId>
            <artifactId>spring-security-jwt</artifactId>
            <version>1.0.9.RELEASE</version>
        </dependency>
         <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-oauth2</artifactId>
        </dependency>
```

<!--more-->

# 认证服务器

## Security配置类

```
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

        manager.createUser(User.withUsername("583188551").password(finalPassword).authorities("USER").build());
        manager.createUser(User.withUsername("724307597").password(finalPassword).authorities("USER").build());
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


    // password 方案三：支持多种编码，通过密码的前缀区分编码方式,推荐
    @Bean
    PasswordEncoder passwordEncoder() {
        return PasswordEncoderFactories.createDelegatingPasswordEncoder();
    }


    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.formLogin().loginPage("/authentication/require")
                .loginProcessingUrl("/authentication/form")
                .and().authorizeRequests()
                .antMatchers("/oauth/authorize").authenticated()
                .antMatchers("/authentication/require", "/authentication/form", "/oauth/exit")
                .permitAll()
                .anyRequest().authenticated()
                .and()
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
        web.ignoring().antMatchers("/resources/**");
    }
}
```

## 授权服务

```
package com.linjingc.authorizationdemo.config;


import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.autoconfigure.condition.ConditionalOnMissingBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.io.ClassPathResource;
import org.springframework.http.HttpMethod;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.core.userdetails.User;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.oauth2.common.DefaultOAuth2AccessToken;
import org.springframework.security.oauth2.common.OAuth2AccessToken;
import org.springframework.security.oauth2.config.annotation.configurers.ClientDetailsServiceConfigurer;
import org.springframework.security.oauth2.config.annotation.web.configuration.AuthorizationServerConfigurerAdapter;
import org.springframework.security.oauth2.config.annotation.web.configuration.EnableAuthorizationServer;
import org.springframework.security.oauth2.config.annotation.web.configurers.AuthorizationServerEndpointsConfigurer;
import org.springframework.security.oauth2.config.annotation.web.configurers.AuthorizationServerSecurityConfigurer;
import org.springframework.security.oauth2.provider.OAuth2Authentication;
import org.springframework.security.oauth2.provider.token.TokenEnhancer;
import org.springframework.security.oauth2.provider.token.TokenEnhancerChain;
import org.springframework.security.oauth2.provider.token.TokenStore;
import org.springframework.security.oauth2.provider.token.store.JwtAccessTokenConverter;
import org.springframework.security.oauth2.provider.token.store.JwtTokenStore;
import org.springframework.security.oauth2.provider.token.store.KeyStoreKeyFactory;

import java.lang.reflect.Array;
import java.util.Arrays;
import java.util.HashMap;
import java.util.Map;

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
    private AuthenticationManager authenticationManager;

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
                .authorizedGrantTypes("password", "authorization_code", "refresh_token")
                .scopes("all")
                .accessTokenValiditySeconds(60)
                .redirectUris("http://my.cloud.com/login")
                .autoApprove(true)
                .refreshTokenValiditySeconds(200);
    }

    /**
     * 配置授权服务器终结点的非安全功能，如令牌存储、令牌自定义、用户批准和授予类型 请求方式
     *
     * @param endpoints
     * @throws Exception
     */
    @Override
    public void configure(AuthorizationServerEndpointsConfigurer endpoints) throws Exception {
        //      将增强的token设置到增强链中
        TokenEnhancerChain tokenEnhancerChain = new TokenEnhancerChain();
        tokenEnhancerChain.setTokenEnhancers(Arrays.asList(jwtTokenEnhancer(),jwtAccessTokenConverter()));

        endpoints
                .tokenStore(tokenStore())
                //.accessTokenConverter(jwtAccessTokenConverter())
                .tokenEnhancer(tokenEnhancerChain)
                .authenticationManager(authenticationManager)
                //允许 GET、POST 请求获取 token，即访问端点：oauth/token
                .allowedTokenEndpointRequestMethods(HttpMethod.GET, HttpMethod.POST)
                //刷新token
                .reuseRefreshTokens(true);
    }

    /**
     * 配置（授权服务器安全配置器安全）
     * 访问tokenkey时需要经过认证
     *
     * @param security
     * @throws Exception
     */
    @Override
    public void configure(AuthorizationServerSecurityConfigurer security) throws Exception {
        //访问tokenkey时需要经过认证
        //security.allowFormAuthenticationForClients().tokenKeyAccess("permitAll()")//公开/oauth/token的接口
        //        .checkTokenAccess("permitAll()")
        security.tokenKeyAccess("permitAll()")         //能够获取token的
                .checkTokenAccess("permitAll()");     //检测是否认证
        //.allowFormAuthenticationForClients();
    }


    /**
     * TokenStore
     *
     * @return
     */
    @Bean
    public TokenStore tokenStore() {
        return new JwtTokenStore(jwtAccessTokenConverter());
    }

    /**
     * 生成JTW token
     *
     * @return
     */
    @Bean
    public JwtAccessTokenConverter jwtAccessTokenConverter() {
        JwtAccessTokenConverter converter = new JwtAccessTokenConverter();
       // 添加证书 非对称性加密
        KeyStoreKeyFactory factory = new KeyStoreKeyFactory(new ClassPathResource("keystore.jks"),"mypass".toCharArray());
        converter.setKeyPair(factory.getKeyPair("mytest"));

//        //对称性加密
//        converter.setSigningKey("linjingc");//生成签名的key
        return converter;
    }


    /**
     * 用于扩展JWT
     *
     * @return
     */
    @Bean
    //@ConditionalOnMissingBean(name = "jwtTokenEnhancer")
    public TokenEnhancer jwtTokenEnhancer() {
        return new MyJwtTokenEnhancer();
    }

    /**
     * 扩展token内容
     */
    public class MyJwtTokenEnhancer implements TokenEnhancer {
        @Override
        public OAuth2AccessToken enhance(OAuth2AccessToken accessToken, OAuth2Authentication authentication) {
            final Map<String, Object> additionalInfo = new HashMap<>();
            User user = (User) authentication.getUserAuthentication().getPrincipal();
            additionalInfo.put("登录名", user.getUsername());
            additionalInfo.put("权限", user.getAuthorities());
            additionalInfo.put("blog", "http://linjingc.top");
            ((DefaultOAuth2AccessToken) accessToken).setAdditionalInformation(additionalInfo);
            return accessToken;
        }
    }
}

```

# 登录和退出的controller

```
package com.linjingc.authorizationdemo.controller;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.security.core.Authentication;
import org.springframework.security.oauth2.common.OAuth2AccessToken;
import org.springframework.security.oauth2.provider.token.DefaultTokenServices;
import org.springframework.security.oauth2.provider.token.TokenStore;
import org.springframework.security.web.authentication.logout.SecurityContextLogoutHandler;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

/**
 * 登录
 * @author cxc
 */
@Controller
public class LoginController {
    @Autowired
    private TokenStore tokenStore;

    @RequestMapping("/")
    @ResponseBody
    public String index() {
        return "进入认证服务器";
    }

    /**
     * 登录页
     */
    @GetMapping("/authentication/require")
    public String login() {
        return "login";
    }

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


    /**
     * 获取当前用户
     * @param user
     * @return
     */
    @ResponseBody
    @RequestMapping(value = "/user")
    public Authentication user(Authentication user) {
        return user;
    }
}

```



# 资源服务器 SSO客户端

## 配置sso客户端

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

退出的话 是请求sso退出 成功后再请求授权服务器 然后授权服务器处理后再回调回来

实现了 删除两边的token 及其退出

## 自定义oauth2 注销方法

```
package com.linjingc.zuuldemo.config;

import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.http.HttpEntity;
import org.springframework.http.HttpHeaders;
import org.springframework.http.HttpMethod;
import org.springframework.http.ResponseEntity;
import org.springframework.http.converter.FormHttpMessageConverter;
import org.springframework.http.converter.HttpMessageConverter;
import org.springframework.http.converter.StringHttpMessageConverter;
import org.springframework.security.core.Authentication;
import org.springframework.security.oauth2.provider.authentication.OAuth2AuthenticationDetails;
import org.springframework.security.web.authentication.logout.LogoutHandler;
import org.springframework.security.web.authentication.logout.SecurityContextLogoutHandler;
import org.springframework.stereotype.Component;
import org.springframework.util.LinkedMultiValueMap;
import org.springframework.util.MultiValueMap;
import org.springframework.web.client.HttpClientErrorException;
import org.springframework.web.client.RestTemplate;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.util.Arrays;

/**
 *  自定义oauth2 注销方法
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

## 配置文件

```
auth-server: http://my.oauth.com:8200
# 退出sso的目标方法地址
auth-server-logout-method: oauth/exit

security:
  oauth2:
    client:
      client-id: client
      client-secret: 123456
      access-token-uri: ${auth-server}/oauth/token
      user-authorization-uri: ${auth-server}/oauth/authorize
    resource:
      jwt:
        key-uri: ${auth-server}/oauth/token_key
```



# 注意点

* 生成的证书就是 keystore.jks 这个文件需要放入到授权服务器的资源文件夹下

* 完成了两种加密 模式 在签名中 非对称加密和对称性加密
  是这样处理的
  客户端启动的时候向授权服务器发送 token_key
  请求 获取公钥
  如果报错 无法启动项目
  如果成功的话 客户端就利用token 来获取

* 需要注意的是 要先启动授权服务器再启动sso客户端
  因为 sso客户端启动的时候会发送一个请求给授权服务器
  返回400就无法启动了 还有一点需要注意
  如果授权服务器更换了秘钥 客户端需要重启 不然会导致公钥匹配不上

* 如果没有配置key uri的话 无法启动 
* 配置错了话 会导致校验不成功 会一直重定向 最终报错 因为没登录成功 又请求了认证服务器 然后认证通过 又回调回来 发现token无法解析 又重新认证 最终导致….

# 对称性加密

对称性加密的话 配置: 授权服务器

```
  @Bean
    public JwtAccessTokenConverter jwtAccessTokenConverter() {
        JwtAccessTokenConverter converter = new JwtAccessTokenConverter();
//        //对称性加密
//        converter.setSigningKey("linjingc");//生成签名的key
        return converter;
    }


```

# 非对称加密

```
  @Bean
    public JwtAccessTokenConverter jwtAccessTokenConverter() {
        JwtAccessTokenConverter converter = new JwtAccessTokenConverter();
       // 添加证书 非对称性加密
        KeyStoreKeyFactory factory = new KeyStoreKeyFactory(new ClassPathResource("keystore.jks"),"mypass".toCharArray());
        converter.setKeyPair(factory.getKeyPair("mytest"));
        return converter;
    }
```

pom添加

```
 <build>
         <resources>
            <resource>
                <directory>src/main/resources</directory>
                <filtering>true</filtering>
                <excludes>
                    <exclude>**/*.jks</exclude>
                </excludes>
            </resource>
            <resource>
                <directory>src/main/resources</directory>
                <filtering>false</filtering>
                <includes>
                    <include>**/*.jks</include>
                </includes>
            </resource>
        </resources>
    </build>
```

## 如何生成私钥和公钥

* ## **生成私钥和证书**

```
keytool -genkeypair -alias serverkey -keyalg RSA -keysize 2048 -validity 3650 -keystore tomatocc.keystore
```

```
参数说明

storepass keystore 文件存储密码
keypass 私钥加解密密码
alias 实体别名(包括证书私钥)
dname 证书个人信息
keyalt 采用公钥算法，默认是DSA
keysize 密钥长度(DSA算法对应的默认算法是sha1withDSA，不支持2048长度，此时需指定RSA)
validity 有效期
keystore 指定keystore文件
```

 在当前目录下就可以看到我们生成的私钥

## **查看keystore详情**

```
我们执行如下命令，然后输入密钥库口令再回车，就可以看到我们的keystore详情
keytool -v -list -keystore tomatocc.keystore
```

