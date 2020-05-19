---
title: mybaitis拦截自定义显示sql
date: 2020-04-13 09:21:22
tags: [java,mybaitis]
---

# mybaitis拦截自定义显示sql

## 配置

```java
@Intercepts({@Signature(type = Executor.class,
        method = "query",
        args = {MappedStatement.class, Object.class, RowBounds.class, ResultHandler.class})})
public class JunliPlugin  implements Interceptor {

    @Override
    public Object intercept(Invocation invocation) throws Throwable {
        MappedStatement mappedStatement = (MappedStatement) invocation.getArgs()[0];
        BoundSql boundSql = mappedStatement.getBoundSql(invocation.getArgs()[1]);
        System.out.println(String.format("plugin output sql = %s , param=%s", boundSql.getSql(),boundSql.getParameterObject()));
        return invocation.proceed();
    }
    @Override
    public Object plugin(Object o) {
        return Plugin.wrap(o,this);
    }
    @Override
    public void setProperties(Properties properties) {

    }
```

<!--more-->

## xml配置插件

```java
 <mappers>
    <mapper resource="xml/TestMapper.xml"/>
    <mapper resource="xml/PostsMapper.xml"/>
</mappers>
```

