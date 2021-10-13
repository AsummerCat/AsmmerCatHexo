---
title: SpringSecurity搭建多入口登录
date: 2021-10-13 22:47:29
tags: [SpringSecurity]
---

# 引入pom.xml
```
<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-security</artifactId>
		</dependency>
```

##  拒绝访问后置类
```
/**
 * 访问拒绝后置处理
 */
@Component
public class CustomAuthenticationAccessDeniedHandler implements AccessDeniedHandler {
    @Override
    public void handle(HttpServletRequest httpServletRequest, HttpServletResponse response, AccessDeniedException e) throws IOException, ServletException {
        response.setContentType("application/json;charset=utf-8");
        response.setStatus(HttpServletResponse.SC_FORBIDDEN);
        PrintWriter writer = response.getWriter();
        ResponseResult<Object> result = new ResponseResult<>(403, "访问被拒绝");
        writer.write(new ObjectMapper().writeValueAsString(result));
        writer.flush();
        writer.close();
    }
}

```
<!--more-->

## 登录失败后置处理
```
/**
 * 登录失败后置处理
 */
@Component
public class CustomAuthenticationEntryPointHandler implements AuthenticationEntryPoint {

    @Override
    public void commence(HttpServletRequest httpServletRequest, HttpServletResponse response, AuthenticationException e) throws IOException, ServletException {
        response.setContentType("application/json;charset=utf-8");
        response.setStatus(HttpServletResponse.SC_FORBIDDEN);
        PrintWriter writer = response.getWriter();
        ResponseResult<Object> result = new ResponseResult<>(401, "请登录");
        writer.write(new ObjectMapper().writeValueAsString(result));
        writer.flush();
        writer.close();
    }
}

```

## 认证失败后置处理
```
/**
 * 认证失败后置处理
 */
@Component("customAuthenticationFailureHandler")
public class CustomAuthenticationFailureHandler extends SimpleUrlAuthenticationFailureHandler {

    @Override
    public void onAuthenticationFailure(HttpServletRequest request, HttpServletResponse response,
                                        AuthenticationException exception) throws IOException {
        response.setContentType("application/json;charset=utf-8");
        response.setStatus(HttpServletResponse.SC_FORBIDDEN);
        PrintWriter writer = response.getWriter();
        ResponseResult<Object> result = new ResponseResult<>(403, "认证失败");
        writer.write(new ObjectMapper().writeValueAsString(result));
        writer.flush();
        writer.close();
    }
}
```

## 注销成功后置处理

```
/**
 * 注销成功后置处理
 */
@Component
public class CustomAuthenticationLogoutSuccessHandler implements LogoutSuccessHandler {

    @Override
    public void onLogoutSuccess(HttpServletRequest httpServletRequest, HttpServletResponse response, Authentication authentication) throws IOException, ServletException {
        response.setContentType("application/json;charset=utf-8");
        response.setStatus(HttpServletResponse.SC_OK);
        PrintWriter writer = response.getWriter();
        writer.write(new ObjectMapper().writeValueAsString(ResponseResult.success("注销成功")));
        writer.flush();
        writer.close();
    }
}
```

## 登录成功后置处理
```
/**
 * 登录成功后置处理
 */
@Component("customAuthenticationSuccessHandler")
public class CustomAuthenticationSuccessHandler extends SavedRequestAwareAuthenticationSuccessHandler {
    @Override
    public void onAuthenticationSuccess(HttpServletRequest request, HttpServletResponse response, Authentication authentication) throws ServletException, IOException {
        response.setContentType("application/json;charset=utf-8");
        response.setStatus(HttpServletResponse.SC_OK);
        PrintWriter writer = response.getWriter();
        writer.write(new ObjectMapper().writeValueAsString(ResponseResult.success("success")));
        writer.flush();
        writer.close();
    }
}
```

## 自定义基础用户
可按照自己需求扩展
```
/**
 * 基础用户信息
 */
@Getter
@Setter
public class BaseUser extends User {
    private Long id;
    private String username;
    private String password;

    public BaseUser(String username, String password, Collection<? extends GrantedAuthority> authorities) {
        super(username, password, authorities);
        this.username = username;
        this.password = password;
    }
}

```

## Security配置
```

/**
 * @author cxc
 * @date 2018/10/16 15:36
 * Security配置类
 */
@Slf4j
@Configuration
@EnableWebSecurity //注解开启Security
@EnableGlobalMethodSecurity(prePostEnabled = true, jsr250Enabled = true)   //开启Security注解  然后在controller中就可以使用方法注解
public class SecurityConfig extends WebSecurityConfigurerAdapter {


    //@PreAuthorize("hasRole('admin')")
    //@PreAuthorize("hasAuthority('update')")
    //@PreAuthorize("hasRole('ROLE_AAA')")

    //(UserDetails) SecurityContextHolder.getContext().getAuthentication() .getPrincipal();
    // BaseUser a= (BaseUser) SecurityContextHolder.getContext().getAuthentication().getPrincipal();
    @Autowired
    private AuthenticationManager authenticationManager;
    @Autowired
    @Qualifier("customUserServiceImpl")
    private UserDetailsService userDetailsService;
    @Autowired
    private CustomAuthenticationSuccessHandler customAuthenticationSuccessHandler;
    @Autowired
    private CustomAuthenticationFailureHandler customAuthenticationFailureHandler;
    @Autowired
    private CustomAuthenticationEntryPointHandler customAuthenticationEntryPointHandler;
    @Autowired
    private CustomAuthenticationLogoutSuccessHandler customAuthenticationLogoutSuccessHandler;
    @Autowired
    private CustomAuthenticationAccessDeniedHandler customAuthenticationAccessDeniedHandler;

    @Autowired
    private FqAuthenticationProvider fqAuthenticationProvider;


    private static final String[] PERMIT_PATHS = new String[]{
            "/attractInvestmentDisk/**",
            "/baseDict/**",
            "/check/**",
            "/constructionTaskFundDisk/**",
            "/indexDisk/**",
            "/industryDict/**",
            "/innovateDisk/**",
            //"/logInfo/**",
            "/matrixDict/**",
            "/projectDisk/**",
            "/yearFundDisk/**",
            "/attribute/**",
            "/department/**",
            "/importantProject/**",
            "/importantProjectDetail/**",
            "/relocateLand/**",
            "/speedProject/**",
    };
    //ingore是完全绕过了spring security的所有filter，相当于不走spring security        忽略css.jq.img等文件
    private static String[] IGNORING_PATHS = new String[]{"/doc.html", "/swagger-resources/**"};


    /**
     * 这里可以设置忽略的路径或者文件
     */
    @Override
    public void configure(WebSecurity web) throws Exception {

        log.info("--------------------------SecurityConfig忽略文件及路径----------------------------");
        //web.ignoring().antMatchers(IGNORING_PATHS);
        web.ignoring().antMatchers(IGNORING_PATHS).antMatchers(PERMIT_PATHS);
    }

    /**
     * 这里是权限控制配置
     */
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        log.info("--------------------------SecurityConfig加载成功----------------------------");

        //关闭csrf跨域攻击防御
        http.csrf().disable();
        /*
         * session 管理   SessionAuthenticationStrategy接口的多个实现
         * 参考 ： https://silentwu.iteye.com/blog/2213956
         * .sessionFixation().changeSessionId() 登录时修改session id 避免会话标识未更新 的漏洞
         * .maximumSessions(5) 最大允许同一用户同时创建5个会话
         *
         */
        http.sessionManagement().sessionCreationPolicy(SessionCreationPolicy.ALWAYS)
                .maximumSessions(1);
        //http.sessionManagement().sessionFixation().changeSessionId().
        //        sessionCreationPolicy(SessionCreationPolicy.IF_REQUIRED)
        //        .maximumSessions(5).sessionRegistry(new SessionRegistryImpl());


        http.httpBasic();
        //配置自定义登录的过滤器
        http.addFilterAfter(fqAuthenticationFilter(), UsernamePasswordAuthenticationFilter.class);


        //设置权限拦截
        http.authorizeRequests()
                //.antMatchers(PERMIT_PATHS).permitAll() //不需要权限访问
                .antMatchers("/").authenticated()   //该路径需要验证通过
                .antMatchers("/lo").access("hasRole('AAA') or hasAuthority('add')") //该路径需要角色  or 权限XXX
                //还有一种方法 就是在方法名称上写注解    @PreAuthorize("hasAnyAuthority('USER','delete')")   //注解拦截权限
                //任何以ROLE_开头的权限都被视为角色`
                .anyRequest().authenticated(); //都要权限  放在最后

        //cookie相关
        //开启cookie保存用户数据
        http.rememberMe()
                //设置cookie有效期
                .tokenValiditySeconds(60 * 60 * 24 * 7);


        //未登录时提示
        http.exceptionHandling().authenticationEntryPoint(customAuthenticationEntryPointHandler)
                //访问拒绝处理，返回json
                .accessDeniedHandler(customAuthenticationAccessDeniedHandler);

        //注销设置
        http.logout()
                .logoutUrl("/logout")
                .invalidateHttpSession(true)
                .logoutSuccessHandler(customAuthenticationLogoutSuccessHandler);

        //关闭原来的login表单
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
        //将两个自定义认证器都注册
        //auth.authenticationProvider(defaultProvider);
        auth.authenticationProvider(fqAuthenticationProvider);
        /**
         * 方式二 数据库查询用户信息
         */
        auth.userDetailsService(userDetailsService).passwordEncoder(passwordEncoder());//添加自定义的userDetailsService认证  //现在已经要加密.passwordEncoder(new MyPasswordEncoder())
        auth.eraseCredentials(false);   //这里是清除还是不清除登录的密码  SecurityContextHolder中
    }


    /**
     * 防止注解使用不了
     */
    @Bean
    @Override
    public AuthenticationManager authenticationManagerBean() throws Exception {
        return super.authenticationManagerBean();
    }

    /**
     * 不加密 官方已经不推荐了
     * 自定义密码加密器
     */
    @Bean
    public static NoOpPasswordEncoder passwordEncoder() {
        return (NoOpPasswordEncoder) NoOpPasswordEncoder.getInstance();
    }

    ///**
    // * BCryptPasswordEncoder 使用BCrypt的强散列哈希加密实现，并可以由客户端指定加密的强度strength，强度越高安全性自然就越高，默认为10.
    // * 自定义密码加密器
    // * BCryptPasswordEncoder(int strength, SecureRandom random)
    // * SecureRandom secureRandom3 = SecureRandom.getInstance("SHA1PRNG");
    // */
    //@Bean
    //public static BCryptPasswordEncoder bCryptPasswordEncoder() {
    //    return new BCryptPasswordEncoder();
    //}
    //
    ///**
    // * StandardPasswordEncoder 1024次迭代的SHA-256散列哈希加密实现，并使用一个随机8字节的salt。
    // * 自定义密码加密器
    // * 盐值不需要用户提供，每次随机生成；
    // * public StandardPasswordEncoder(CharSequence secret) 可以设置一个秘钥值
    // * 计算方式: 迭代SHA算法+密钥+随机盐来对密码加密，加密后得到的密码是80位
    // */
    //@Bean
    //public static StandardPasswordEncoder standardPasswordEncoder() {
    //    return new StandardPasswordEncoder();
    //}

    @Bean
    public FqAuthenticationFilter fqAuthenticationFilter() {
        FqAuthenticationFilter filter = new FqAuthenticationFilter();
        filter.setAuthenticationManager(authenticationManager);
        filter.setAuthenticationSuccessHandler(customAuthenticationSuccessHandler);
        filter.setAuthenticationFailureHandler(customAuthenticationFailureHandler);
        return filter;
    }

```
如果需要多入口配置
需要配置:  
1.`configure(AuthenticationManagerBuilder auth) `中注册`auth.authenticationProvider(fqAuthenticationProvider);`

2.并且创建一个自定义过滤器
```
 @Bean
    public FqAuthenticationFilter fqAuthenticationFilter() {
        FqAuthenticationFilter filter = new FqAuthenticationFilter();
        filter.setAuthenticationManager(authenticationManager);
        filter.setAuthenticationSuccessHandler(customAuthenticationSuccessHandler);
        filter.setAuthenticationFailureHandler(customAuthenticationFailureHandler);
        return filter;
    }
```
3.在`void configure(HttpSecurity http)`中加入
```
 //配置自定义登录的过滤器
        http.addFilterAfter(fqAuthenticationFilter(), UsernamePasswordAuthenticationFilter.class);
```



## 多入口配置编写 例子
### FqAuthenticationFilter
```

public class FqAuthenticationFilter extends AbstractAuthenticationProcessingFilter {

    public static final String SPRING_SECURITY_FORM_USERNAME_KEY = "username";

    public static final String SPRING_SECURITY_FORM_PASSWORD_KEY = "password";

    //从前台传过来的参数
    private String usernameParameter = SPRING_SECURITY_FORM_USERNAME_KEY;

    private String passwordParameter = SPRING_SECURITY_FORM_PASSWORD_KEY;

    private boolean postOnly = true;

    public FqAuthenticationFilter() {
        super(new AntPathRequestMatcher("/fq/login", "POST"));
    }

    /**
     * 执行实际身份验证。实现应执行以下操作之一：
     * 1、为经过身份验证的用户返回填充的身份验证令牌，表示身份验证成功
     * 2、返回null，表示认证过程还在进行中。 在返回之前，实现应该执行完成流程所需的任何额外工作。
     * 3、如果身份验证过程失败，则抛出AuthenticationException
     */
    @Override
    public Authentication attemptAuthentication(HttpServletRequest request, HttpServletResponse response)
            throws AuthenticationException {
        if (this.postOnly && !request.getMethod().equals("POST")) {
            throw new AuthenticationServiceException("Authentication method not supported: " + request.getMethod());
        }
        String username = obtainUsername(request);
        username = (username != null) ? username : "";
        username = username.trim();
        String password = obtainPassword(request);
        password = (password != null) ? password : "";

        BaseUser user = new BaseUser(username, password, new ArrayList<>());
        FqAuthenticationToken authRequest = new FqAuthenticationToken(user, password);
        // 可以放一些其他信息进去
        setDetails(request, authRequest);
        return this.getAuthenticationManager().authenticate(authRequest);
    }

    @Nullable
    protected String obtainPassword(HttpServletRequest request) {
        return request.getParameter(this.passwordParameter);
    }

    @Nullable
    protected String obtainUsername(HttpServletRequest request) {
        return request.getParameter(this.usernameParameter);
    }

    protected void setDetails(HttpServletRequest request, FqAuthenticationToken authRequest) {
        authRequest.setDetails(this.authenticationDetailsSource.buildDetails(request));
    }

}
```

### FqAuthenticationProvider

```

@Slf4j
@Component
public class FqAuthenticationProvider implements AuthenticationProvider {

    @Autowired
    private LoginService userService;
    @Autowired
    private PasswordEncoder passwordEncoder;

    /**
     * 认证
     */
    @Override
    public Authentication authenticate(Authentication authentication) throws AuthenticationException {
        if (!supports(authentication.getClass())) {
            return null;
        }
        FqAuthenticationToken token = (FqAuthenticationToken) authentication;
        BaseUser principal = (BaseUser) token.getPrincipal();
        //查询用户数据
        BaseUser user = userService.loadUserByUsername(principal.getUsername());
        if (user == null) {
            throw new InternalAuthenticationServiceException("无法获取用户信息");
        }

        String presentedPassword = authentication.getCredentials().toString();
        if (!passwordEncoder.matches(presentedPassword, user.getPassword())) {
            throw new InternalAuthenticationServiceException("Authentication failed: password does not match stored value");
        }
        //将密码隐藏起来
        user.setPassword(null);
        FqAuthenticationToken result = new FqAuthenticationToken(user, null, user.getAuthorities());
                /*
                Details 中包含了 ip地址、 sessionId 等等属性 也可以存储一些自己想要放进去的内容
                */
        result.setDetails(token.getDetails());
        return result;
    }

    @Override
    public boolean supports(Class<?> aClass) {
        return FqAuthenticationToken.class.isAssignableFrom(aClass);
    }
}
```

### FqAuthenticationToken
```
public class FqAuthenticationToken extends AbstractAuthenticationToken {

    private static final long serialVersionUID = SpringSecurityCoreVersion.SERIAL_VERSION_UID;

    // 这里指的账号密码哈
    private final Object principal;

    private Object credentials;

    /**
     * 没经过身份验证时，初始化权限为空，setAuthenticated(false)设置为不可信令牌
     */
    public FqAuthenticationToken(Object principal, Object credentials) {
        super(null);
        this.principal = principal;
        this.credentials = credentials;
        setAuthenticated(false);
    }

    /**
     * 经过身份验证后，将权限放进去，setAuthenticated(true)设置为可信令牌
     */
    public FqAuthenticationToken(Object principal, Object credentials,
                                 Collection<? extends GrantedAuthority> authorities) {
        super(authorities);
        this.principal = principal;
        this.credentials = credentials;
        super.setAuthenticated(true); // must use super, as we override
    }

    @Override
    public Object getCredentials() {
        return this.credentials;
    }

    @Override
    public Object getPrincipal() {
        return this.principal;
    }

    @Override
    public void setAuthenticated(boolean isAuthenticated) throws IllegalArgumentException {
        Assert.isTrue(!isAuthenticated,
                "Cannot set this token to trusted - use constructor which takes a GrantedAuthority list instead");
        super.setAuthenticated(false);
    }

    @Override
    public void eraseCredentials() {
        super.eraseCredentials();
        this.credentials = null;
    }

}

```

这样就构建完成了
