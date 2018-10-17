---
title: SpringBoot整合shiro(新)
date: 2018-10-12 19:11:10
tags: [SpringBoot,shiro]
---
#[demo地址](https://github.com/AsummerCat/SpringBootAndShiroNow)  

---

>参考: 
>1.[Spring Boot Shiro 权限信息缓存处理,记住我，thymleaf使用shiro标签](https://blog.csdn.net/u014695188/article/details/52356158)  
>2.[https://blog.csdn.net/xtiawxf/article/details/52571949](https://blog.csdn.net/xtiawxf/article/details/52571949)  

<!--more-->

# 前言

>集成Shiro核心内容:
>
>1.ShiroFilterFactory，Shiro过滤器工程类，具体的实现类是：ShiroFilterFactoryBean，此实现类是依赖于SecurityManager安全管理器。主要配置Filter就好。

<!--more-->

>2.SecurityManager，Shiro的安全管理，主要是身份认证的管理，缓存管理，cookie管理，所以在实际开发中我们主要是和SecurityManager进行打交道的。
>  
>3.Realm,用于身份信息权限信息的验证。开发时集成AuthorizingRealm，重写两个方法:doGetAuthenticationInfo(获取即将需要认真的信息)、doGetAuthorizationInfo(获取通过认证后的权限信息)。
>
>4.HashedCredentialsMatcher，凭证匹配器，用于告诉Shiro在认证时通过什么方式(算法)来匹配密码。默认(storedCredentialsHexEncoded=false)Base64编码，可以修改为(storedCredentialsHexEncoded=true)Hex编码。
>
>5.LifecycleBeanPostProcessor，Shiro生命周期处理器，保证实现了Shiro内部lifecycle函数的bean执行。

>6.开启Shiro的注解功能(如@RequiresRoles,@RequiresPermissions),需借助SpringAOP扫描使用Shiro注解的类,并在必要时进行安全逻辑验证，需要配置两个bean(DefaultAdvisorAutoProxyCreator(可选)和AuthorizationAttributeSourceAdvisor)实现此功能。
>
>7.其它的就是缓存管理，记住登录、验证码、分布式系统共享Session之类的，这些大部分都是需要自己进行的实现，其中缓存管理，记住登录比较简单实现，并需要注入到SecurityManager让Shiro的安全管理器进行管理就好了。后续章节中会一 一补充。

# 目录  
0.shiro常见的异常类  
1.导入相关的pom.xml  
2.创建用户登录的实体  
3.创建自定义授权登录类  
4.application.properties中添加 关于shiro的配置  
5.加盐测试类  
6.编写自定义密码匹配类  
7.创建shiro生命周期配置类(单独)  
8.创建shiro配置类(完整)  
9.shiro配置类中加入 thymeleaf 在页面使用shiro的标签的bean  
10.shiro配置类中加入 实现 代码中使用注解  
11.在shiro中加入EhCache缓存 缓存用户权限信息  
12.shiro使用缓存来存储用户验证时密码多次输入错误  
13.记住密码  
14.注意事项

---

```
// 全局获取shiro中的当前用户
SecurityUtils.getSubject().getPrincipal() 
```

---

# shiro常见的异常类

```
(验证)
authc:
AuthencationException:
AuthenticationException 异常是Shiro在登录认证过程中，认证失败需要抛出的异常。

AuthenticationException包含以下子类：

    CredentitalsException 凭证异常

        IncorrectCredentialsException 不正确的凭证

        ExpiredCredentialsException 凭证过期

    AccountException 账号异常

        ConcurrentAccessException 并发访问异常（多个用户同时登录时抛出）

        UnknownAccountException 未知的账号

        ExcessiveAttemptsException 认证次数超过限制

        DisabledAccountException 禁用的账号
            LockedAccountException 账号被锁定

    UnsupportedTokenException 使用了不支持的Token

    

###############################################################################################


authz:
AuthorizationException:
子类:
    UnauthorizedException:抛出以指示请求的操作或对请求的资源的访问是不允许的。
    UnanthenticatedException:当尚未完成成功认证时，尝试执行授权操作时引发异常。
    (授权只能在成功的认证之后执行，因为授权数据（角色、权限等）必须总是与已知的标识相关联。这样的已知身份只能在成功登录时获得。)

```



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
    <version>1.4.0</version>
</dependency>

```

<!--more-->

然后再导入 页面整合shiro的相关内容

```
<dependency>
     <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
    </dependency>
    
    <!-- 在Thymeleaf模板引擎中集成Shiro -->
    <dependency>
    		<groupId>com.github.theborakompanioni</groupId>
    		<artifactId>thymeleaf-extras-shiro</artifactId>
    		<version>1.2.1</version>
    </dependency>	
```

其他: 

```
<!-- 包含支持UI模版（Velocity，FreeMarker，JasperReports）， 邮件服务， 脚本服务(JRuby)， 缓存Cache（EHCache）， 任务计划Scheduling（uartz）。 -->
       <dependency>
          <groupId>org.springframework</groupId>
          <artifactId>spring-context-support</artifactId>
      </dependency>

      <dependency>
          <groupId>org.projectlombok</groupId>
          <artifactId>lombok</artifactId>
          <optional>true</optional>
      </dependency>
```

---
# 创建一个用户登录的实体

需要注意的是 `实体需要序列化`

```
package com.linjingc.demo.vo;

import lombok.Data;
import lombok.ToString;

import java.io.Serializable;
import java.util.Set;

/**
 * @author cxc
 * @date 2018/10/12 09:16
 */
@Data
@ToString
public class User implements Serializable {
    private static final long serialVersionUID = 1L;
    private String username;
    private String password;
    private Integer age;
    private Set roles;
    private String salt;
}


```

---

# 创建自定义授权登录类

`AuthRealm`完成根据用户名去数据库的查询,并且将用户信息放入shiro中,供第二个类调用.`CredentialsMatcher`,完成对于密码的校验.其中用户的信息来自shiro.AuthRealm类如下:

```
package com.linjingc.demo.shiro;

import com.linjingc.demo.service.UserService;
import com.linjingc.demo.vo.User;
import lombok.extern.slf4j.Slf4j;
import org.apache.shiro.authc.*;
import org.apache.shiro.authz.AuthorizationInfo;
import org.apache.shiro.authz.SimpleAuthorizationInfo;
import org.apache.shiro.realm.AuthorizingRealm;
import org.apache.shiro.subject.PrincipalCollection;
import org.apache.shiro.util.ByteSource;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;

import java.util.ArrayList;
import java.util.List;
import java.util.Set;

/**
 * @author cxc
 * shiro自定义授权类
 */
@Slf4j
public class AuthRealm extends AuthorizingRealm {
    @Autowired
    private UserService userService;
    @Value("${shiro.salt}")
    private String shiroSalt;

    //认证.登录
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {
        //UsernamePasswordToken对象用来存放提交的登录信息 登录表单
        UsernamePasswordToken utoken = (UsernamePasswordToken) token;
        String username = utoken.getUsername();

        //实际项目中，这里可以根据实际情况做缓存，如果不做，Shiro自己也是有时间间隔机制，2分钟内不会重复执行该方法
        User user = userService.findUserByUserName(username);

        //用户是否存在
        if (user == null) {
            throw new UnknownAccountException();
        }

        //加盐 可以自定义可以尝试  userName+salt
        ByteSource salt = ByteSource.Util.bytes(user.getUsername() + shiroSalt);

        //放入Shiro.调用CredentialsMatcher检验密码
        return new SimpleAuthenticationInfo(user, user.getPassword(), salt, getName());
    }

    //授权
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principal) {
        //获取session中的用户
        User user = (User) principal.getPrimaryPrincipal();
        log.info("当前用户授权中");
        List<String> permissions = new ArrayList<>();
        List<String> userRoles = new ArrayList<>();

        //用户角色
        Set roles = user.getRoles();
        if (roles.size() > 0) {
            //将查询到的用户权限放入一个权限集合中
            permissions.addAll(roles);
            //将查询到的用户角色放入一个角色集合中
            userRoles.add("root");
        }
        SimpleAuthorizationInfo info = new SimpleAuthorizationInfo();
        //将权限放入shiro中.
        info.addStringPermissions(permissions);
        //将用户角色放入session
        info.addRoles(userRoles);
        log.info("当前用户授权成功");
        return info;
    }
}
```

---

# 在application.properties中添加 关于shiro的配置

```
#自定义shiro加密类型 散列算法:MD5,sha-1,sha-256
shiro.encrypt.type=MD5
#自定义shiro加密次数
shiro.HashIterations=1024
#自定义shiro加盐的通用部分
shiro.salt=ABCDEFG
#自定义密码错误上限
shiro.password.error.size=5
#shiro记住我的session最大时长 秒
shiro.rememberMe.Max.time=259200

#关闭thymeleaf页面缓存
spring.thymeleaf.cache=false

```
---

# 加盐测试类

```
package com.linjingc.demo.shiro;

import org.apache.shiro.crypto.hash.SimpleHash;
import org.apache.shiro.util.ByteSource;

/**
 * @author cxc
 * @date 2018/10/12 11:42
 * 加盐测试类
 */
public class SaltUtil {
    /**
     * 加盐测试类
     *
     * @param args
     */
    public static void main(String[] args) {
        String user = "admin";
        String hashAlgorithmName = "MD5";//加密方式
        Object crdentials = "a123456";//密码原值
        ByteSource salt = ByteSource.Util.bytes(user + "ABCDEFG");//以账号作为盐值+ABCDEFG
        int hashIterations = 1024;//加密1024次
        Object result = new SimpleHash(hashAlgorithmName, crdentials, salt, hashIterations);
        System.out.println(user + ":" + result);
    }
}

```

---

# 编写自定义密码匹配类

**三种方式**

 继承SimpleCredentialsMatcher 实现自定义加密
 
## 自定义凭证匹配器 然后 直接对比密码是否跟库里相等

```
package com.linjingc.demo.shiro;

import lombok.extern.slf4j.Slf4j;
import org.apache.shiro.authc.AuthenticationInfo;
import org.apache.shiro.authc.AuthenticationToken;
import org.apache.shiro.authc.UsernamePasswordToken;
import org.apache.shiro.authc.credential.SimpleCredentialsMatcher;

/**
 * @author cxc
 * 自定义密码校验器
 * 简单密码对比校验
 */
@Slf4j
public class CredentialsMatcher extends SimpleCredentialsMatcher {

    /**
     * 自定义密码校验器
     */
    @Override
    public boolean doCredentialsMatch(AuthenticationToken token, AuthenticationInfo info) {

        log.info("进入自定义密码校验器");
        UsernamePasswordToken utoken = (UsernamePasswordToken) token;
        //获得用户输入的密码:(可以采用加盐(salt)的方式去检验)
        String inPassword = new String(utoken.getPassword());
        //获得数据库中的密码
        String dbPassword = (String) info.getCredentials();
        //进行密码的比对
        boolean flag = this.equals(inPassword, dbPassword);
        return flag;
    }


}
```

## 自定义凭证匹配器 然后根据加盐来匹配

```
package com.linjingc.demo.shiro;

import lombok.extern.slf4j.Slf4j;
import org.apache.shiro.authc.AuthenticationInfo;
import org.apache.shiro.authc.AuthenticationToken;
import org.apache.shiro.authc.UsernamePasswordToken;
import org.apache.shiro.authc.credential.SimpleCredentialsMatcher;
import org.apache.shiro.crypto.hash.SimpleHash;
import org.springframework.beans.factory.annotation.Value;

/**
 * @author cxc
 * 自定义密码校验器
 * 加盐校验
 */
@Slf4j
public class CredentialsSaltMatcher extends SimpleCredentialsMatcher {
    @Value("${shiro.encrypt.type}")
    private String encryptType;
    @Value("${shiro.HashIterations}")
    private int hashIterations;
    @Value("${shiro.salt}")
    private String shiroSalt;

    /**
     * 自定义密码校验器
     */
    @Override
    public boolean doCredentialsMatch(AuthenticationToken token, AuthenticationInfo info) {

        log.info("进入自定义密码加盐校验器");
        UsernamePasswordToken utoken = (UsernamePasswordToken) token;
        //获得用户输入的密码:(可以采用加盐(salt)的方式去检验)
        String inPassword = new SimpleHash(encryptType, utoken.getPassword(), utoken.getUsername() + shiroSalt, hashIterations).toString();

        //获得数据库中的密码
        String dbPassword = (String) info.getCredentials();

        //进行密码的比对
        boolean flag = this.equals(inPassword, dbPassword);
        return flag;
    }
}
```

## 直接写在ShiroConfig中  稍后创建

```
@Bean
    public HashedCredentialsMatcher hashedCredentialsMatcher() {
        HashedCredentialsMatcher hashedCredentialsMatcher = new HashedCredentialsMatcher();
        //storedCredentialsHexEncoded默认是true，此时用的是密码加密用的是Hex编码；false时用Base64编码
        hashedCredentialsMatcher.setStoredCredentialsHexEncoded(true);
        //散列算法:md5,sha-1,sha-256
        hashedCredentialsMatcher.setHashAlgorithmName(encryptType);
        //散列次数
        hashedCredentialsMatcher.setHashIterations(hashIterations);
        return hashedCredentialsMatcher;
    }
```

---

# 创建shiro生命周期配置类(单独)

**如果不单独创建的话,放在shiro配置类中,可能会导致无注入value**

```
package com.linjingc.demo.shiro;

import org.apache.shiro.spring.LifecycleBeanPostProcessor;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * @author cxc
 * @date 2018/10/12 15:52
 * shiro生命周期处理器(单独)
 * 如果放在shiroConfig中可能会导致@value无法注入 所以单独放在一个配置类中
 */
@Configuration
public class ShiroInit {
    /**
     * Shiro生命周期处理器
     *
     * @return
     */
    @Bean
    public LifecycleBeanPostProcessor lifecycleBeanPostProcessor() {
        return new LifecycleBeanPostProcessor();
    }
}

```
---

# 创建shiro配置类(完整)

```
package com.linjingc.demo.shiro;

import at.pollux.thymeleaf.shiro.dialect.ShiroDialect;
import lombok.extern.slf4j.Slf4j;
import org.apache.shiro.authc.credential.HashedCredentialsMatcher;
import org.apache.shiro.cache.ehcache.EhCacheManager;
import org.apache.shiro.mgt.SecurityManager;
import org.apache.shiro.spring.security.interceptor.AuthorizationAttributeSourceAdvisor;
import org.apache.shiro.spring.web.ShiroFilterFactoryBean;
import org.apache.shiro.web.mgt.CookieRememberMeManager;
import org.apache.shiro.web.mgt.DefaultWebSecurityManager;
import org.apache.shiro.web.servlet.SimpleCookie;
import org.springframework.aop.framework.autoproxy.DefaultAdvisorAutoProxyCreator;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.DependsOn;

import java.util.LinkedHashMap;
import java.util.Map;

/**
 * @author cxc
 * shiro配置类
 */
@Configuration
@Slf4j
public class ShiroConfig {

    @Value("${shiro.encrypt.type}")
    private String encryptType;
    @Value("${shiro.HashIterations}")
    private int hashIterations;
    @Value("${shiro.rememberMe.Max.time}")
    private int rememberMeMaxTime;


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
        // 必须设置 SecurityManager 核心
        shiroFilterFactoryBean.setSecurityManager(securityManager);

        // 如果不设置默认会自动寻找Web工程根目录下的"/login.jsp"页面
        //打开页面跳转的地址
        shiroFilterFactoryBean.setLoginUrl("/login");
        // 登录成功后要跳转的链接
        shiroFilterFactoryBean.setSuccessUrl("/index");
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

        //配置记住我的访问路径
        filterChainDefinitionMap.put("/success", "user");

        //配置权限
        filterChainDefinitionMap.put("/lo1", "perms[add]");
        filterChainDefinitionMap.put("/lo2", "perms[update]");
        filterChainDefinitionMap.put("/lo3", "perms[delete]");
        filterChainDefinitionMap.put("/lo4", "perms[User],perms[AAA]");
        filterChainDefinitionMap.put("/lo5", "authc,roles[管理员],perms[AAA]");
        // 这里为了测试，固定写死的值，也可以从数据库或其他配置中读取

        //需要通过认证
        filterChainDefinitionMap.put("/hello", "authc");


        // <!-- 过滤链定义，从上向下顺序执行，一般将 /**放在最为下边 -->:这是一个坑呢，一不小心代码就不好使了;
        // <!-- authc:所有url都必须认证通过才可以访问; anon:所有url都都可以匿名访问-->
        filterChainDefinitionMap.put("/**", "authc");

        //添加拦截器
        shiroFilterFactoryBean.setFilterChainDefinitionMap(filterChainDefinitionMap);
        log.info("--------------------------Shiro拦截器工厂类注入成功------------------------------------");
        return shiroFilterFactoryBean;
    }


    /**
     * 调用自定义方法
     * 安全管理器：securityManager 核心部分
     */
    @Bean(name = "securityManager")
    public SecurityManager securityManager(@Qualifier("authRealm") AuthRealm authRealm) {
        log.info("--------------------------shiro已经加载------------------------------------");
        DefaultWebSecurityManager manager = new DefaultWebSecurityManager();
        //设置realm
        manager.setRealm(authRealm);
        //注入缓存管理器;
        //这个如果执行多次，也是同样的一个对象;
        manager.setCacheManager(ehCacheManager());
        //注入记住我管理器
        manager.setRememberMeManager(rememberMeManager());
        return manager;
    }

    /**
     * shiro缓存管理器;
     * 需要注入对应的其它的实体类中：
     * EhCacheManager，缓存管理，用户登陆成功后，把用户信息和权限信息缓存起来，
     * 然后每次用户请求时，放入用户的session中，如果不设置这个bean，每个请求都会查询一次数据库。
     */
    @Bean(name = "ehCacheManager")
    //控制bean加载顺序 表示被注解的bean在初始化时,指定的bean需要先完成初始化。
    @DependsOn("lifecycleBeanPostProcessor")
    public EhCacheManager ehCacheManager() {
        log.info("------------------shiro缓存注入成功-------------------------------");
        EhCacheManager cacheManager = new EhCacheManager();
        cacheManager.setCacheManagerConfigFile("classpath:config/ehcache-shiro.xml");
        return cacheManager;
    }


    /**
     * 身份认证realm; (这个需要自己写，账号密码校验；权限等)
     * 配置自定义的权限登录器
     *
     * @param matcher 参数是密码比较器,注入到authRealm中 以下自定义凭证匹配器三选一注入
     */
    @Bean(name = "authRealm")
    public AuthRealm authRealm(HashedCredentialsMatcher matcher) {
        AuthRealm authRealm = new AuthRealm();
        authRealm.setCredentialsMatcher(matcher);
        return authRealm;
    }


    //配置自定义的凭证匹配器
    //方式一 加盐
    @Bean
    public HashedCredentialsMatcher hashedCredentialsMatcher() {
        //自定义的凭证匹配器 可以用来记录密码错误次数
        HashedCredentialsMatcher hashedCredentialsMatcher = new RetryLimitHashedCredentialsMatcher(ehCacheManager());
        //原先的自定义凭证匹配器
        //HashedCredentialsMatcher hashedCredentialsMatcher = new HashedCredentialsMatcher();

        //storedCredentialsHexEncoded默认是true，此时用的是密码加密用的是Hex编码；false时用Base64编码
        hashedCredentialsMatcher.setStoredCredentialsHexEncoded(true);
        //散列算法:md5,sha-1,sha-256
        hashedCredentialsMatcher.setHashAlgorithmName(encryptType);
        //散列次数
        hashedCredentialsMatcher.setHashIterations(hashIterations);
        return hashedCredentialsMatcher;
    }

    //方式二 加密
    @Bean
    public CredentialsSaltMatcher credentialsSaltMatcher() {
        CredentialsSaltMatcher credentialsSaltMatcher = new CredentialsSaltMatcher();
        return credentialsSaltMatcher;
    }

    //方式三 自定义加密
    @Bean
    public CredentialsMatcher credentialsMatcher() {
        CredentialsMatcher credentialsMatcher = new CredentialsMatcher();
        return credentialsMatcher;
    }


    /**
     * 开启Shiro的注解(如@RequiresRoles,@RequiresPermissions),需借助SpringAOP扫描使用Shiro注解的类,并在必要时进行安全逻辑验证
     * 配置以下两个bean(DefaultAdvisorAutoProxyCreator(可选)和AuthorizationAttributeSourceAdvisor)即可实现此功能
     */

    /**
     * 开启shiro aop注解支持.
     * 使用代理方式;所以需要开启代码支持;
     *
     * @param securityManager
     * @return
     */
    @Bean
    public AuthorizationAttributeSourceAdvisor authorizationAttributeSourceAdvisor(SecurityManager securityManager) {
        AuthorizationAttributeSourceAdvisor authorizationAttributeSourceAdvisor = new AuthorizationAttributeSourceAdvisor();
        authorizationAttributeSourceAdvisor.setSecurityManager(securityManager);
        return authorizationAttributeSourceAdvisor;
    }


    @Bean
    @DependsOn({"lifecycleBeanPostProcessor"})
    public DefaultAdvisorAutoProxyCreator advisorAutoProxyCreator() {
        DefaultAdvisorAutoProxyCreator advisorAutoProxyCreator = new DefaultAdvisorAutoProxyCreator();
        advisorAutoProxyCreator.setProxyTargetClass(true);
        return advisorAutoProxyCreator;
    }


    /**
     * 添加ShiroDialect 为了在thymeleaf里使用shiro的标签的bean
     */
    @Bean(name = "shiroDialect")
    public ShiroDialect shiroDialect() {
        return new ShiroDialect();
    }


    /**
     * 记住我
     */
    @Bean
    public SimpleCookie rememberMeCookie() {

        log.info("--------------------------shiro的记住我功能加载成功--------------------------");
        //这个参数是cookie的名称，对应前端的checkbox的name = rememberMe
        SimpleCookie simpleCookie = new SimpleCookie("rememberMe");
        //<!-- 记住我cookie生效时间30天 ,单位秒;-->
        simpleCookie.setMaxAge(rememberMeMaxTime);
        return simpleCookie;
    }

	/**
     * Cookie管理对象
     */
    @Bean
    public CookieRememberMeManager rememberMeManager() {

        log.info("--------------------------shiroCookie管理对象加载成功--------------------------");
        CookieRememberMeManager cookieRememberMeManager = new CookieRememberMeManager();
        cookieRememberMeManager.setCookie(rememberMeCookie());
        return cookieRememberMeManager;
    }
}

```

---

# shiro配置类中加入 thymeleaf 在页面使用shiro的标签的bean

```
    /**
     * 添加ShiroDialect 为了在thymeleaf里使用shiro的标签的bean
     */
    @Bean(name = "shiroDialect")
    public ShiroDialect shiroDialect() {
        return new ShiroDialect();
    }
```

页面:  
`<html xmlns:shiro="http://www.pollix.at/thymeleaf/shiro">`

使用:
>可以参考 [thymeleaf-extras-shiro 看标签使用](https://github.com/theborakompanioni/thymeleaf-extras-shiro)

```
基本语法:

<shiro:principal/>        表示获取这个用户的对象

<shiro:principal property="username" />        表示获取这用户的属性

<shiro:guest=">       验证当前用户是否为“访客” 未登录。

<shiro:user="">        认证通过或已记住的用户

<shiro:authenticated="">      当前用户认证通过 和 未记住用户 

<shiro:notAuthenticated="">   当前用户认证通过   和 已记住用户  

<shiro:hasRole="admin" >      判断当前用户拥有这个角色

<p shiro:lacksRole="developer">     判断当前用户没有这个角色

<p shiro:hasAllRoles="developer, admin">     判断当前用户是否拥有这些角色所有

<p shiro:hasAnyRoles="foo, bar">      判断当前用户是否拥有这些角色之一

<p shiro:hasPermission="userInfo:add">     判断用户是否拥有当前指定权限

<p shiro: lacksPermission ="userInfo:add">    判断用户是否不拥有当前指定权限

<p shiro: hasAllPermissions ="userInfo:add">    判断用户是否拥有当前所有权限

<p shiro: hasAnyPermissions ="userInfo:add">    判断用户是否拥有当前权限之一


```

案例:  

```
<!DOCTYPE html>  
<html xmlns="http://www.w3.org/1999/xhtml" xmlns:th="http://www.thymeleaf.org"
      xmlns:shiro="http://www.pollix.at/thymeleaf/shiro">
<head>  
<meta charset="UTF-8" />  
<title>Insert title here</title>  
</head>  
<body>  
    <h3>index</h3> 
    <!-- 验证当前用户是否为“访客”，即未认证（包含未记住）的用户。 -->
    <p shiro:guest="">Please <a href="login.html">login</a></p>
    
    
    <!-- 认证通过或已记住的用户。 -->
    <p shiro:user="">
       Welcome back John! Not John? Click <a href="login.html">here</a> to login.
    </p>
    
    <!-- 已认证通过的用户。不包含已记住的用户，这是与user标签的区别所在。 -->
    <p shiro:authenticated="">
      Hello, <span shiro:principal=""></span>, how are you today?
    </p> 
    <a shiro:authenticated="" href="updateAccount.html">Update your contact information</a>
    
    <!-- 输出当前用户信息，通常为登录帐号信息。 -->
    <p>Hello, <shiro:principal/>, how are you today?</p>
    
    
    <!-- 未认证通过用户，与authenticated标签相对应。与guest标签的区别是，该标签包含已记住用户。 -->
    <p shiro:notAuthenticated="">
       Please <a href="login.html">login</a> in order to update your credit card information.
    </p>
     
    <!-- 验证当前用户是否属于该角色。 -->
    <a shiro:hasRole="admin" href="admin.html">Administer the system</a><!-- 拥有该角色 -->
    
    <!-- 与hasRole标签逻辑相反，当用户不属于该角色时验证通过。 -->
    <p shiro:lacksRole="developer"><!-- 没有该角色 -->
      Sorry, you are not allowed to developer the system.
    </p>
    
    <!-- 验证当前用户是否属于以下所有角色。 -->
    <p shiro:hasAllRoles="developer, admin"><!-- 角色与判断 -->
       You are a developer and a admin.
    </p>
    
    <!-- 验证当前用户是否属于以下任意一个角色。  -->
    <p shiro:hasAnyRoles="admin, vip, developer"><!-- 角色或判断 -->
         You are a admin, vip, or developer.
    </p>
    
    <!--验证当前用户是否拥有指定权限。  -->
    <a shiro:hasPermission="userInfo:add" href="createUser.html">添加用户</a><!-- 拥有权限 -->
    
    <!-- 与hasPermission标签逻辑相反，当前用户没有制定权限时，验证通过。 -->
    <p shiro:lacksPermission="userInfo:del"><!-- 没有权限 -->
         Sorry, you are not allowed to delete user accounts.
    </p>
    
    <!-- 验证当前用户是否拥有以下所有权限。 -->
    <p shiro:hasAllPermissions="userInfo:view, userInfo:add"><!-- 权限与判断 -->
           You can see or add users.
    </p>
    
    <!-- 验证当前用户是否拥有以下任意一个权限。  -->
    <p shiro:hasAnyPermissions="userInfo:view, userInfo:del"><!-- 权限或判断 -->
               You can see or delete users.
    </p>
    
</body>  
</html>

```

---

# shiro配置类中加入 实现 代码中使用注解

开启Shiro的注解(如@RequiresRoles,@RequiresPermissions)

```
 /**
     * 开启Shiro的注解(如@RequiresRoles,@RequiresPermissions),需借助SpringAOP扫描使用Shiro注解的类,并在必要时进行安全逻辑验证
     * 配置以下两个bean(DefaultAdvisorAutoProxyCreator(可选)和AuthorizationAttributeSourceAdvisor)即可实现此功能
     */
    
    /**
     *  开启shiro aop注解支持.
     *  使用代理方式;所以需要开启代码支持;
     * @param securityManager
     * @return
     */
    @Bean
    public AuthorizationAttributeSourceAdvisor authorizationAttributeSourceAdvisor(SecurityManager securityManager) {
        AuthorizationAttributeSourceAdvisor authorizationAttributeSourceAdvisor = new AuthorizationAttributeSourceAdvisor();
        authorizationAttributeSourceAdvisor.setSecurityManager(securityManager);
        return authorizationAttributeSourceAdvisor;
    }
    
    
    @Bean
    @DependsOn({"lifecycleBeanPostProcessor"})
    public DefaultAdvisorAutoProxyCreator advisorAutoProxyCreator() {
        DefaultAdvisorAutoProxyCreator advisorAutoProxyCreator = new DefaultAdvisorAutoProxyCreator();
        advisorAutoProxyCreator.setProxyTargetClass(true);
        return advisorAutoProxyCreator;
    }
```

使用:

```
@RequiresAuthentication   表示当前Subject已经通过login 进行了身份验证；即Subject. isAuthenticated()返回true。  

@RequiresUser   表示当前Subject已经身份验证或者通过记住我登录的。

@RequiresGuest   　　表示当前Subject没有身份验证或通过记住我登录过，即是游客身份。

@RequiresRoles(value={“admin”, “user”}, logical= Logical.AND)
@RequiresRoles(value={“admin”})
@RequiresRoles({“admin“})
表示当前Subject需要角色admin 和user。
 
@RequiresPermissions (value={“user:a”, “user:b”}, logical= Logical.OR)
表示当前Subject需要权限user:a或user:b。

既可以用在controller中，也可以用在service中
建议将shiro注解放入controller，因为如果service层使用了spring的事物注解，那么shiro注解将无效
```



---


# 在shiro中加入EhCache缓存 缓存用户权限信息

>作用:  
>实际中我们的权限信息是不怎么会改变的，所以我们希望是第一次访问，然后进行缓存处理，那么Shiro是否支持呢，答案是肯定的，Shiro中加入缓存机制。  

  
>简单来说 就是每次需要验证权限信息的时候都要查询一次 现在就直接丢进缓存中去 


## 导入pom.xml  看顶部标题 导入pom的那部分
## 在shiro配置类中加入EhCache缓存器

```
/**
     * shiro缓存管理器;
     * 需要注入对应的其它的实体类中：
     * EhCacheManager，缓存管理，用户登陆成功后，把用户信息和权限信息缓存起来，
     * 然后每次用户请求时，放入用户的session中，如果不设置这个bean，每个请求都会查询一次数据库。
     */
    @Bean(name = "ehCacheManager")
    //控制bean加载顺序 表示被注解的bean在初始化时,指定的bean需要先完成初始化。
    @DependsOn("lifecycleBeanPostProcessor")
    public EhCacheManager ehCacheManager() {
        log.info("------------------shiro缓存注入成功-------------------------------");
        EhCacheManager cacheManager = new EhCacheManager();
        cacheManager.setCacheManagerConfigFile("classpath:config/ehcache-shiro.xml");
        return cacheManager;
    }
```

## 在resources中添加关于EhCache的配置文件

路径: `config/ehcache-shiro.xml`

```
<?xml version="1.0" encoding="UTF-8"?>
<ehcache name="es">
    <diskStore path="java.io.tmpdir"/>
    <!--
  name:缓存名称。
  maxElementsInMemory:缓存最大数目
  maxElementsOnDisk：硬盘最大缓存个数。
  eternal:对象是否永久有效，一但设置了，timeout将不起作用。
  overflowToDisk:是否保存到磁盘，当系统当机时
  timeToIdleSeconds:设置对象在失效前的允许闲置时间（单位：秒）。仅当eternal=false对象不是永久有效时使用，可选属性，默认值是0，也就是可闲置时间无穷大。
  timeToLiveSeconds:设置对象在失效前允许存活时间（单位：秒）。最大时间介于创建时间和失效时间之间。仅当eternal=false对象不是永久有效时使用，默认是0.，也就是对象存活时间无穷大。
  diskPersistent：是否缓存虚拟机重启期数据 Whether the disk store persists between restarts of the Virtual Machine. The default value is false.
  diskSpoolBufferSizeMB：这个参数设置DiskStore（磁盘缓存）的缓存区大小。默认是30MB。每个Cache都应该有自己的一个缓冲区。
  diskExpiryThreadIntervalSeconds：磁盘失效线程运行时间间隔，默认是120秒。
  memoryStoreEvictionPolicy：当达到maxElementsInMemory限制时，Ehcache将会根据指定的策略去清理内存。默认策略是LRU（最近最少使用）。你可以设置为FIFO（先进先出）或是LFU（较少使用）。
  clearOnFlush：内存数量最大时是否清除。
  memoryStoreEvictionPolicy:
       Ehcache的三种清空策略;
       FIFO，first in first out，这个是大家最熟的，先进先出。
       LFU， Less Frequently Used，就是上面例子中使用的策略，直白一点就是讲一直以来最少被使用的。如上面所讲，缓存的元素有一个hit属性，hit值最小的将会被清出缓存。
       LRU，Least Recently Used，最近最少使用的，缓存的元素有一个时间戳，当缓存容量满了，而又需要腾出地方来缓存新的元素的时候，那么现有缓存元素中时间戳离当前时间最远的元素将被清出缓存。
-->
    <defaultCache
            maxElementsInMemory="10000"
            eternal="false"
            timeToIdleSeconds="120"
            timeToLiveSeconds="120"
            overflowToDisk="false"
            diskPersistent="false"
            diskExpiryThreadIntervalSeconds="120"
    />
    <!-- 登录记录缓存锁定10分钟 -->
    <cache name="passwordRetryCache"
           maxEntriesLocalHeap="2000"
           eternal="false"
           timeToIdleSeconds="3600"
           timeToLiveSeconds="0"
           overflowToDisk="false"
           statistics="true">
    </cache>
</ehcache>

```

## 开启的Ehcache加入到shiro的安全管理器中

`就是那个我们自定义的securityManager`

```
        //注入缓存管理器;
        //这个如果执行多次，也是同样的一个对象;
        manager.setCacheManager(ehCacheManager());
    
```
这样就开启了缓存

---

# shiro使用缓存来存储用户验证时密码多次输入错误

>CredentialsMatcher是shiro提供的用于加密密码和验证密码服务的接口，而HashedCredentialsMatcher正是CredentialsMatcher的一个实现类
>

所以我们需要修改 原先的自定义自定义凭证匹配器 在里面加入关于用户验证密码

## 创建一个自定义的类 继承 `HashedCredentialsMatcher`

```
package com.linjingc.demo.shiro;

import org.apache.shiro.authc.AuthenticationInfo;
import org.apache.shiro.authc.AuthenticationToken;
import org.apache.shiro.authc.ExcessiveAttemptsException;
import org.apache.shiro.authc.credential.HashedCredentialsMatcher;
import org.apache.shiro.cache.Cache;
import org.apache.shiro.cache.CacheManager;
import org.springframework.beans.factory.annotation.Value;

import java.util.concurrent.atomic.AtomicInteger;

/**
 * @author cxc
 * 自定义密码校验器误次数
 */
public class RetryLimitHashedCredentialsMatcher extends HashedCredentialsMatcher {
    private Cache<String, AtomicInteger> passwordRetryCache;

    @Value("${shiro.password.error.size}")
    private int passwordErrorSize;

    public RetryLimitHashedCredentialsMatcher(CacheManager cacheManager) {
        //用于记录缓存的名称 passwordRetryCache
        passwordRetryCache = cacheManager.getCache("passwordRetryCache");
    }

    @Override
    public boolean doCredentialsMatch(AuthenticationToken token,
                                      AuthenticationInfo info) {
        String username = (String) token.getPrincipal();
        // retry count + 1  
        AtomicInteger retryCount = passwordRetryCache.get(username);
        if (retryCount == null) {
            retryCount = new AtomicInteger(0);
            passwordRetryCache.put(username, retryCount);
        }
        if (retryCount.incrementAndGet() > passwordErrorSize) {
            // if retry count > 5 throw
            //超出定义的错误次数后 抛出一个异常
            throw new ExcessiveAttemptsException();
        }

        boolean matches = super.doCredentialsMatch(token, info);
        if (matches) {
            // clear retry count  
            passwordRetryCache.remove(username);
        }
        return matches;
    }

}

```
在回调方法`doCredentialsMatch(AuthenticationToken token,AuthenticationInfo info)`中进行身份认证的密码匹配，这里我们引入了`Ehcahe`用于保存用户登录次数，如果登录失败`retryCount`变量则会一直累加，如果登录成功，那么这个`count`就会从缓存中移除，从而实现了如果登录次数超出指定的值就锁定。

**这样到达上限后 会抛出`throw ExcessiveAttemptsException()` 这样我们可以在登录的方法中捕获抛出自定义提示**

## 修改shiroConfig中的自定义密码校验器

将原先的直接new的 `HashedCredentialsMatcher` 改成我们自定义的`RetryLimitHashedCredentialsMatcher `

```
 //配置自定义的凭证匹配器
    //方式一 加盐
    @Bean
    public HashedCredentialsMatcher hashedCredentialsMatcher() {
        //自定义的凭证匹配器 可以用来记录密码错误次数
        HashedCredentialsMatcher hashedCredentialsMatcher = new RetryLimitHashedCredentialsMatcher(ehCacheManager());
        //原先的自定义凭证匹配器
        //HashedCredentialsMatcher hashedCredentialsMatcher = new HashedCredentialsMatcher();
        
        //storedCredentialsHexEncoded默认是true，此时用的是密码加密用的是Hex编码；false时用Base64编码
        hashedCredentialsMatcher.setStoredCredentialsHexEncoded(true);
        //散列算法:md5,sha-1,sha-256
        hashedCredentialsMatcher.setHashAlgorithmName(encryptType);
        //散列次数
        hashedCredentialsMatcher.setHashIterations(hashIterations);
        return hashedCredentialsMatcher;
    }

```


## 在Ehcache.xml中添加关于密码错误次数的缓存

```
<!-- 登录记录缓存锁定10分钟 -->
    <cache name="passwordRetryCache"
           maxEntriesLocalHeap="2000"
           eternal="false"
           timeToIdleSeconds="3600"
           timeToLiveSeconds="0"
           overflowToDisk="false"
           statistics="true">
    </cache>

```

## 在application.properties中添加

```
#自定义密码错误上限
shiro.password.error.size=5
```
---

# 记住密码
## 加入两个方法

```
 /**
     * 记住我
     */
    @Bean
    public SimpleCookie rememberMeCookie() {

        log.info("--------------------------shiro的记住我功能加载成功--------------------------");
        //这个参数是cookie的名称，对应前端的checkbox的name = rememberMe
        SimpleCookie simpleCookie = new SimpleCookie("rememberMe");
        //<!-- 记住我cookie生效时间30天 ,单位秒;-->
        simpleCookie.setMaxAge(rememberMeMaxTime);
        return simpleCookie;
    }

    /**
     * Cookie管理对象
     */
    @Bean
    public CookieRememberMeManager rememberMeManager() {

        log.info("--------------------------shiroCookie管理对象加载成功--------------------------");
        CookieRememberMeManager cookieRememberMeManager = new CookieRememberMeManager();
        cookieRememberMeManager.setCookie(rememberMeCookie());
        return cookieRememberMeManager;
    }

```
## 加入到安全管理器中

```
       //注入记住我管理器
	    manager.setRememberMeManager(rememberMeManager());
```

## 然后再Shirofiter过滤器中添加 记住我的路径

```
 //配置记住我的访问路径
        filterChainDefinitionMap.put("/success", "user");
```

## 登录界面加入rememberMe复选框

`    <div><label> 记住我 : <input type="checkbox" name="rememberMe"/> </label></div>
`

## 使用
这样关闭浏览器还是可以访问的

这时候运行程序，登录之后跳转到/index页面，然后我们关闭浏览器，然后直接访问/index还是可以访问的，说明我们写的记住密码已经生效了，但是只能访问 过滤器中user类型的路径 其他还是需要登陆的 

---

# 注意事项

# 启动时报错 【Springboot+Themeleaf模板+Shiro标签】找不到类AbstractProcessorDialect解决
>使用的SpringBoot1.5.2.RELEASE版本集成Thymeleaf时，它使用的版本是2.1.5.RELEASE，而在这个版本中没有`AbstractProcessorDialect`类。

解决方法一：可以把Thymeleaf版本更改为3.0.7.RELEASE  
`<thymeleaf.version>3.0.7.RELEASE</thymeleaf.version> `  
再加上  
`<thymeleaf-layout-dialect.version>2.2.2</thymeleaf-layout-dialect.version>`

解决方法二：还可以把thymeleaf-extras-shiro的版本改为1.2.1

---

# 遇到@value() 无法注入到配置类的情况

>参考[BeanPostProcessor加载次序及其对Bean造成的影响分析](https://blog.csdn.net/m0_37962779/article/details/78605478)  

```
解决方案:

将 LifecycleBeanPostProcessor 单独抽出到一个@Configuration
```

# 如果权限验证类中 的认证出错的话

页面的shiro: XXX 标签是无法使用的

# 登录登出的时候org.apache.shiro.crypto.CryptoException: Unable to execute 'doFinal' with cipher instance

参考: [看这里](https://blog.csdn.net/qq_21727627/article/details/72850391)

---
