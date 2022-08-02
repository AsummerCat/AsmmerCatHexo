---
title: Cat客户端集成springboot三
date: 2022-08-02 11:58:03
tags: [链路跟踪,cat]
---
# Cat客户端集成springboot

## 引入pom
```
<dependency>
    <groupId>com.dianping.cat</groupId>
    <artifactId>cat-client</artifactId>
    <version>${cat.version}</version>
</dependency>
```
<!--more-->

## 配置文件
<!--more-->
需要在你的项目中创建 `src/main/resources/META-INF/app.properties` 文件, 并添加如下内容:
项目名称(仅支持英文或者数字)
```
app.name={appkey}
```

## 编写过滤器
```
/**
 * 请求监控拦截
 * @author tengx
 */
public class CatServletFilter implements Filter {
   
 
    @Override
    public void init(FilterConfig filterConfig) {
 
    }
 
    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
 
        HttpServletRequest request = (HttpServletRequest) servletRequest;
        CatContext catContext = getCatContext(request);
        Cat.logRemoteCallServer(catContext);
        //开启一个事务, 类型为url, 名称为借口路径
        Transaction t = Cat.newTransaction(CatConstants.TYPE_URL, request.getRequestURL().toString());
        try {
          //记录事件日志
            catLogEvent(request);
            filterChain.doFilter(servletRequest, servletResponse);
            //成功执行Transaction
            t.setStatus(Transaction.SUCCESS);
        } catch (Exception ex) {
        //失败设置异常
            t.setStatus(ex);
            Cat.logError(ex);
            throw ex;
        } finally {
        //关闭当前事务
            t.complete();
        }
    }
 
    /**
     * 获取CAT监控上下文环境
     * @param request 请求
     * @return CAT上下文环境
     */
    private CatContext getCatContext(HttpServletRequest request) {
        CatContext catContext = new CatContext();
        catContext.addProperty(Cat.Context.ROOT, request.getHeader(CatHttpConstants.CAT_HTTP_HEADER_ROOT_MESSAGE_ID));
        catContext.addProperty(Cat.Context.PARENT, request.getHeader(CatHttpConstants.CAT_HTTP_HEADER_PARENT_MESSAGE_ID));
        catContext.addProperty(Cat.Context.CHILD, request.getHeader(CatHttpConstants.CAT_HTTP_HEADER_CHILD_MESSAGE_ID));
        return catContext;
    }
 
    /**
     * 使用CAT event事件记录信息
     * @param request 请求
     */
    private void catLogEvent(HttpServletRequest request) {
        Cat.logEvent(CatHttpConstants.SERVER_METHOD, request.getMethod());
        Cat.logEvent(CatHttpConstants.SERVER_CLIENT, request.getRemoteHost());
        Cat.logEvent(CatHttpConstants.SERVER_REFERER, request.getHeader("referer"));
        Cat.logEvent(CatHttpConstants.SERVER_AGENT, request.getHeader("user-agent"));
        Cat.logEvent(CatHttpConstants.SERVER_TOKEN, request.getHeader("token"));
        Cat.logEvent(CatHttpConstants.SERVER_PARAMS, JsonUtils.toJSONString(request.getParameterMap()));
        Cat.logEvent(CatHttpConstants.SERVER_DATETIME, DateUtil.now());
    }
 
    @Override
    public void destroy() {
 
    }
}
```
这样即可整合springboot

## 使用boot的自动加载机制 加载
这里就要知道spring boot 的自动加载机制了。通过在MEAT-INF 下配置spring.factories 文件。在文件中配置自动加载的类。这样当工程引用了这个jar包的时候spring boot 就会自动加载指定的类。代码如下
```
/**
 * 对URL进行拦截的类
 * @author tengx
 */
 
@Configuration
public class CatFilterConfigure {
 
    @Bean("catFilter")
    public Filter createFilter(){
        return new CatServletFilter();
    }
 
    @Bean
    public FilterRegistrationBean catFilter(Filter catFilter) {
        FilterRegistrationBean registration = new FilterRegistrationBean();
        registration.setFilter(catFilter);
        registration.addUrlPatterns("/*");
        registration.setName("cat-filter");
        registration.setOrder(1);
        return registration;
    }
}
```