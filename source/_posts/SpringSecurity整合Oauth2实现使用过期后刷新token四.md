---
title: SpringSecurity整合Oauth2实现使用过期后刷新token四
date: 2019-07-11 22:25:18
tags: [Security,SpringCloud,Oauth2]
---

# SpringSecurity整合Oauth2实现使用过期后刷新token四

# 在认证服务器上添加

```
@Override
    public void configure(AuthorizationServerEndpointsConfigurer endpoints) throws Exception {
        // 为解决获取token并发问题


        endpoints
                .tokenStore(new RedisTokenStore(redisConnectionFactory))
                .authenticationManager(authenticationManager)
                //允许 GET、POST 请求获取 token，即访问端点：oauth/token
                .allowedTokenEndpointRequestMethods(HttpMethod.GET, HttpMethod.POST);

        // 为解决获取token并发问题
        DefaultTokenServices tokenServices = new DefaultTokenServices();
        tokenServices.setTokenStore(endpoints.getTokenStore());
        //这里表示可以使用刷新token
        tokenServices.setSupportRefreshToken(true);
        tokenServices.setClientDetailsService(endpoints.getClientDetailsService());
        tokenServices.setTokenEnhancer(endpoints.getTokenEnhancer());
        endpoints.tokenServices(tokenServices);

    }
```

<!--more-->

这里面的大概逻辑就是

```
创建一个DefaultTokenServices  添加 setSupportRefreshToken属性为true 表示为可以刷新token
默认的情况下是关闭的
```



## 请求刷新的路径

```
url: http://localhost:8100/oauth/token
跟认证的一样
参数的话: refresh_token    xxxx
         grant_type       refresh_token     (这里是固定的)
         client_id        xxxx
         client_secret    xxxx 
         
         
         返回内容:
         {
    "access_token": "2a3e11ae-27fd-43c9-8137-dc288b180aca",
    "token_type": "bearer",
    "refresh_token": "dd8f1420-6a70-497f-8c7f-852f0d2508d9",
    "expires_in": 29,
    "scope": "select"
}

```

我们可以在访问资源服务器的时候 如果报401错误 进行过期重新处理