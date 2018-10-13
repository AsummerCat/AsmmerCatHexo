---
title: SpringBoot整合shiro(旧)
date: 2018-10-11 13:11:10
tags: [SpringBoot,shiro]
---

# 导入pom.xml
导入shiro相关

```
<!-- siro安全框架 -->
     <!-- shiro spring. -->
<dependency>
       <groupId>org.apache.shiro</groupId>
       <artifactId>shiro-spring</artifactId>
       <version>1.4.0</version>
</dependency>
     <!-- shiro ehcache -->
<dependency>
    <groupId>org.apache.shiro</groupId>
    <artifactId>shiro-ehcache</artifactId>
    <version>1.2.2</version>
</dependency>

```

<!--more-->

然后再导入 页面的相关内容

```
<dependency>
     <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
    </dependency>
```


---


# 创建一个shiro的配置类 注入到spring中

```
package com.linjngc;

import org.apache.shiro.authc.credential.CredentialsMatcher;
import org.apache.shiro.mgt.SecurityManager;
import org.apache.shiro.spring.web.ShiroFilterFactoryBean;
import org.apache.shiro.web.mgt.DefaultWebSecurityManager;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.util.LinkedHashMap;
import java.util.Map;

/**
 * @author cxc
 */
@Configuration
public class ShiroConfig {
    /**
     * ShiroFilterFactoryBean 处理拦截资源文件问题。
     * 注意：单独一个ShiroFilterFactoryBean配置是或报错的，以为在
     * 初始化ShiroFilterFactoryBean的时候需要注入：SecurityManager
     * <p>
     * Filter Chain定义说明 1、一个URL可以配置多个Filter，使用逗号分隔 2、当设置多个过滤器时，全部验证通过，才视为通过
     * 3、部分过滤器可指定参数，如perms，roles
     */
    @Bean
    public ShiroFilterFactoryBean shirFilter(SecurityManager securityManager) {
        ShiroFilterFactoryBean shiroFilterFactoryBean = new ShiroFilterFactoryBean();
        // 必须设置 SecurityManager
        shiroFilterFactoryBean.setSecurityManager(securityManager);
        // 如果不设置默认会自动寻找Web工程根目录下的"/login.jsp"页面
        shiroFilterFactoryBean.setLoginUrl("/login");   //打开页面跳转的地址
        // 登录成功后要跳转的链接
        shiroFilterFactoryBean.setSuccessUrl("/");
        // 未授权界面;
        shiroFilterFactoryBean.setUnauthorizedUrl("/403");
        // 拦截器.
        Map<String, String> filterChainDefinitionMap = new LinkedHashMap<String, String>();
        // 配置不会被拦截的链接 顺序判断
        filterChainDefinitionMap.put("/static/**", "anon");
        filterChainDefinitionMap.put("/index", "anon");
        filterChainDefinitionMap.put("/login", "anon");
        filterChainDefinitionMap.put("/loginUser", "anon");

        // 配置退出过滤器,其中的具体的退出代码Shiro已经替我们实现了
        filterChainDefinitionMap.put("/logout", "logout");


        //配置权限
        filterChainDefinitionMap.put("/lo1", "perms[add]");
        filterChainDefinitionMap.put("/lo2", "perms[update]");
        filterChainDefinitionMap.put("/lo3", "perms[delete]");
        filterChainDefinitionMap.put("/lo4", "perms[User],perms[AAA]");
        filterChainDefinitionMap.put("/lo5", "authc,roles[管理员],perms[AAA]");

        //需要通过认证
        filterChainDefinitionMap.put("/hello", "authc");


        // <!-- 过滤链定义，从上向下顺序执行，一般将 /**放在最为下边 -->:这是一个坑呢，一不小心代码就不好使了;
        // <!-- authc:所有url都必须认证通过才可以访问; anon:所有url都都可以匿名访问-->
        filterChainDefinitionMap.put("/**", "authc");
        shiroFilterFactoryBean.setFilterChainDefinitionMap(filterChainDefinitionMap);
        System.err.println("--------------Shiro拦截器工厂类注入成功----------------");
        return shiroFilterFactoryBean;
    }


    /**
     * 调用自定义方法
     */
    @Bean(name = "securityManager")
    public SecurityManager securityManager(@Qualifier("authRealm") AuthRealm authRealm) {
        System.err.println("--------------shiro已经加载----------------");
        DefaultWebSecurityManager manager = new DefaultWebSecurityManager();
        manager.setRealm(authRealm);
        return manager;
    }

    /**
     * 身份认证realm; (这个需要自己写，账号密码校验；权限等)
     * 配置自定义的权限登录器
     *
     * @return
     */
    @Bean(name = "authRealm")
    public AuthRealm authRealm(@Qualifier("credentialsMatcher") CredentialsMatcher matcher) {
        AuthRealm authRealm = new AuthRealm();
        authRealm.setCredentialsMatcher(matcher);
        return authRealm;
    }

    //配置自定义的密码比较器
    @Bean(name = "credentialsMatcher")
    public CredentialsMatcher credentialsMatcher() {
        return new com.linjngc.CredentialsMatcher();
    }
}
```


---


# 配置自定义密码校验器

需要继承`extends SimpleCredentialsMatcher`

```
package com.linjngc;

import org.apache.shiro.authc.AuthenticationInfo;
import org.apache.shiro.authc.AuthenticationToken;
import org.apache.shiro.authc.UsernamePasswordToken;
import org.apache.shiro.authc.credential.SimpleCredentialsMatcher;

public class CredentialsMatcher extends SimpleCredentialsMatcher {

    /**
     * 自定义密码校验器
     */
    @Override
    public boolean doCredentialsMatch(AuthenticationToken token, AuthenticationInfo info) {
        UsernamePasswordToken utoken=(UsernamePasswordToken) token;
        //获得用户输入的密码:(可以采用加盐(salt)的方式去检验)
        String inPassword = new String(utoken.getPassword());
        //获得数据库中的密码
        String dbPassword=(String) info.getCredentials();
        //进行密码的比对
        boolean flag= this.equals(inPassword, dbPassword);
        return flag;
    }
    
}
```

---


# 创建权限控制shiro授权类

```
package com.linjngc;

import org.apache.shiro.SecurityUtils;
import org.apache.shiro.authc.*;
import org.apache.shiro.authz.AuthorizationInfo;
import org.apache.shiro.authz.SimpleAuthorizationInfo;
import org.apache.shiro.realm.AuthorizingRealm;
import org.apache.shiro.subject.PrincipalCollection;
import org.apache.shiro.util.ByteSource;

import java.util.ArrayList;
import java.util.List;


/**
 * shiro授权类
 */
public class AuthRealm extends AuthorizingRealm {
    //@Autowired
    //private UserService userService;
    
    //认证.登录
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {
        UsernamePasswordToken utoken=(UsernamePasswordToken) token;//获取用户输入的token
        String username = utoken.getUsername(); //
        System.out.println(" *****你输入的账号:"+utoken.getUsername());
        User user =new User();
        user.setUsername("ad");    //模拟查询
        user.setPassword("a123456");
        user.setId(999L);

        String salt="ABCDEFG";  //盐

        if (null == user) {
            throw new AccountException("帐号或密码不正确！");
        }
        return new SimpleAuthenticationInfo(user, user.getPassword(), ByteSource.Util.bytes(salt),getName());//放入shiro.调用CredentialsMatcher检验密码
    }
    //授权
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principal) {
        User token = (User) SecurityUtils.getSubject().getPrincipal();   //登录成功的用户
        SimpleAuthorizationInfo info =  new SimpleAuthorizationInfo();
        List<String> permissions=new ArrayList<>();  //权限

        // Set<Role> roles = user.getRoles();
        //if(roles.size()>0) {
        //    for(Role role : roles) {
        //        Set<Module> modules = role.getModules();
        //        if(modules.size()>0) {
        //            for(Module module : modules) {
        //                permissions.add(module.getMname());
        //            }
        //        }
        //    }
        //}
        permissions.add("add");//这里添加权限 可以添加多个权限
        permissions.add("update");
        permissions.add("delete");
        permissions.add("USER");



        List<String> roles=new ArrayList<>();  //角色
                    roles.add("管理员");


        info.addStringPermissions(permissions);//将权限放入shiro中.
        info.addRoles(roles);
        return info;
    }
}
```

---

# 注意

`SecurityUtils.getSubject().getPrincipal();`  
这个可以获取出 保存在shiro的用户session

具体实现看github [SpringBoot整合shiro](https://github.com/AsummerCat/springbootandshiro)