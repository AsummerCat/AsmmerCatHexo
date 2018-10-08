---
title: SpringBoot整合redis
date: 2018-10-08 17:00:15
tags: [SpringBoot,redis]
---

# 1.导入pom.xml
>参考:  
>1.[Spring缓存注解@Cache,@CachePut , @CacheEvict，@CacheConfig使用](https://blog.csdn.net/sanjay_f/article/details/47372967)  
>2.[]()  
>
```
      <!-- 缓存依赖 -->
      <dependency>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-starter-cache</artifactId>
      </dependency>
      <!-- spring boot redis 依赖 -->
      <dependency>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-starter-redis</artifactId>
      </dependency>
```

<!--more-->


---


# 2. 在启动类上加入注解

`@EnableCaching : 开启SpringBoot缓存策略，放在启动主类。`

---

# 3.application.properties 配置

#### 3.1.单机
```
# Redis 配置(默认配置)
# Redis 数据库索引（默认为0）
spring.redis.database=0
# Redis 服务器地址
spring.redis.host=localhost
# Redis 服务器端口
spring.redis.port=6379
# Redis 服务器密码(默认为空)
spring.redis.password=linjingc
# 连接池最大连接数（使用负值表示没有限制）
spring.redis.pool.max-active=8
# 连接池中的最大空闲连接
spring.redis.pool.max-idle=8
# 连接池中的最小空闲连接
spring.redis.pool.min-idle=0
# 连接池最大阻塞等待时间（使用负值表示没有限制）
spring.redis.pool.max-wait=-1
# 设置连接超时
spring.redis.timeout=0

```

#### 3.2集群模式

```
spring:
    application:
        name: spring-boot-redis
    redis:
        host: 192.168.145.132
        port: 6379
        timeout: 20000
        cluster:
            nodes: 192.168.211.134:7000,192.168.211.134:7001,192.168.211.134:7002
            maxRedirects: 6
        pool:
            max-active: 8
            min-idle: 0
            max-idle: 8
            max-wait: -1
```

---



# 4.关于 SpringBoot 缓存注解

```
在支持 Spring Cache 的环境下，


@EnableCaching : 开启SpringBoot缓存策略，放在启动主类。
@CacheConfig(cacheNames = "XXX") : 设置一个名为”XXX”的缓存空间。
@Cacheable : Spring在每次执行前都会检查Cache中是否存在相同key的缓存元素，如果存在就不再执行该方法，而是直接从缓存中获取结果进行返回，否则才会执行并将返回结果存入指定的缓存中。
@CacheEvict : 清除缓存。
@CachePut : @CachePut也可以声明一个方法支持缓存功能。使用@CachePut标注的方法在执行前不会去检查缓存中是否存在之前执行过的结果，而是每次都会执行该方法，并将执行结果以键值对的形式存入指定的缓存中。

```

---

# 5.新建一个redis简单工具类

```
package springboot.config;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.cache.CacheManager;
import org.springframework.cache.annotation.CachingConfigurerSupport;
import org.springframework.cache.annotation.EnableCaching;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.cache.RedisCacheManager;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.core.StringRedisTemplate;
import org.springframework.data.redis.serializer.Jackson2JsonRedisSerializer;

import com.fasterxml.jackson.annotation.JsonAutoDetect;
import com.fasterxml.jackson.annotation.PropertyAccessor;
import com.fasterxml.jackson.databind.ObjectMapper;


/**
 * Redis缓存配置类
 * @author cxc
 *
 */
@Configuration
@EnableCaching
public class RedisConfig extends CachingConfigurerSupport{

    @Value("${spring.redis.host}")
    private String host;
    @Value("${spring.redis.port}")
    private int port;
    @Value("${spring.redis.timeout}")
    private int timeout;
    
    //自定义缓存key生成策略
//    @Bean
//    public KeyGenerator keyGenerator() {
//        return new KeyGenerator(){
//            @Override
//            public Object generate(Object target, java.lang.reflect.Method method, Object... params) {
//                StringBuffer sb = new StringBuffer();
//                sb.append(target.getClass().getName());
//                sb.append(method.getName());
//                for(Object obj:params){
//                    sb.append(obj.toString());
//                }
//                return sb.toString();
//            }
//        };
//    }
    //缓存管理器
    @Bean 
    public CacheManager cacheManager(@SuppressWarnings("rawtypes") RedisTemplate redisTemplate) {
        RedisCacheManager cacheManager = new RedisCacheManager(redisTemplate);
        //设置缓存过期时间 
        cacheManager.setDefaultExpiration(10000);
        return cacheManager;
    }
    @Bean
    public RedisTemplate<String, String> redisTemplate(RedisConnectionFactory factory){
        StringRedisTemplate template = new StringRedisTemplate(factory);
        setSerializer(template);//设置序列化工具
        template.afterPropertiesSet();
        return template;
    }
     private void setSerializer(StringRedisTemplate template){
            @SuppressWarnings({ "rawtypes", "unchecked" })
            Jackson2JsonRedisSerializer jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer(Object.class);
            ObjectMapper om = new ObjectMapper();
            om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
            om.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
            jackson2JsonRedisSerializer.setObjectMapper(om);
            template.setValueSerializer(jackson2JsonRedisSerializer);
     }
}
```

---

# 6.编写相关的实体类

这里注意一定要实现序列化接口用于序列化！

....随便写个vo类

---

# 7.使用

>接下来就是如何使用注解啦,这一步反而是最简单的.其实只用到了两个注解,@Cacheable和@CacheEvict.第一个注解代表从缓存中查询指定的key,如果有,从缓存中取,不再执行方法.如果没有则执
行方法,并且将方法的返回值和指定的key关联起来,放入到缓存中.而@CacheEvict则是从缓存中清除指定的key对应的数据.使用的代码如下:

```
@Cacheable(value="thisredis", key="'users_'+#id")
    public User findUser(Integer id) {
        User user = new User();
        user.setUsername("hlhdidi");
        user.setPassword("123");
        user.setUid(id.longValue());
        System.out.println("log4j2坏啦?");
        logger.info("输入user,用户名:{},密码:{}",user.getUsername(),user.getPassword());
        return user;
    }

    @CacheEvict(value="thisredis", key="'users_'+#id",condition="#id!=1")
    public void delUser(Integer id) {
        // 删除user
        System.out.println("user删除");
    }
```

---


# 8.注解详解

#### 8.1 @Cacheable
@Cacheable的属性的意义  
**cacheNames:这个会在redis中出现一个zset集合的数组用来记录已经缓存的数据**  
cacheNames：指定缓存的名称  
key：定义组成的key值，如果不定义，则使用全部的参数计算一个key值。可以使用spring El表达式

```
/**
     * cacheNames 设置缓存的值 
     *  key：指定缓存的key，这是指参数id值。 key可以使用spEl表达式
     * @param id
     * @return
     */
    @Cacheable(cacheNames="book1", key="#id")
    public Book queryBookCacheable(String id){
        logger.info("queryBookCacheable,id={}",id);
        return repositoryBook.get(id);
    }

    /**
     * 这里使用另一个缓存存储缓存
     * 
     * @param id
     * @return
     */
    @Cacheable(cacheNames="book2", key="#id")
    public Book queryBookCacheable_2(String id){
        logger.info("queryBookCacheable_2,id={}",id);
        return repositoryBook.get(id);
    }
    
     /**
     * 这里使用另一个缓存存储缓存(类似记录变量)
     *   #p0 第一个参数
     * @param id
     * @return
     */
    @Cacheable(cacheNames="book2", key="#p0)")
    public Book queryBookCacheable_2(String id){
        logger.info("queryBookCacheable_2,id={}",id);
        return repositoryBook.get(id);
    }
    

    /**
     * 缓存的key也可以指定对象的成员变量
     * @param qry
     * @return
     */
    @Cacheable(cacheNames="book1", key="#qry.id")
    public Book queryBookCacheableByBookQry(BookQry qry){
        logger.info("queryBookCacheableByBookQry,qry={}",qry);
        String id = qry.getId();
        Assert.notNull(id, "id can't be null!");
        String name = qry.getName();
        Book book = null;
        if(id != null){
            book = repositoryBook.get(id);
            if(book != null && !(name != null && book.getName().equals(name))){
                book = null;
            }
        }
        return book;
    }
    
    
    /***
     * 如果设置sync=true，
     *  如果缓存中没有数据，多个线程同时访问这个方法，则只有一个方法会执行到方法，其它方法需要等待
     *  如果缓存中已经有数据，则多个线程可以同时从缓存中获取数据
     * @param id
     * @return
     */
    @Cacheable(cacheNames="book3", sync=true)
    public Book queryBookCacheableWithSync(String id) {
        logger.info("begin ... queryBookCacheableByBookQry,id={}",id);
        try {
            Thread.sleep(1000 * 2);
        } catch (InterruptedException e) {
        }
        logger.info("end ... queryBookCacheableByBookQry,id={}",id);
        return repositoryBook.get(id);
    }



```

#### 8.2 @CacheEvict

>删除缓存  
allEntries = true: 清空缓存book1里的所有值  
allEntries = false: 默认值，此时只删除key对应的值

```
  /**
     * allEntries = true: 清空book1里的所有缓存
     */
    @CacheEvict(cacheNames="book1", allEntries=true)
    public void clearBook1All(){
        logger.info("clearAll");
    }
    /**
     * 对符合key条件的记录从缓存中book1移除
     */
    @CacheEvict(cacheNames="book1", key="#id")
    public void updateBook(String id, String name){
        logger.info("updateBook");
        Book book = repositoryBook.get(id);
        if(book != null){
            book.setName(name);
            book.setUpdate(new Date());
        }
    }

```

#### 7.3 @CachePut

每次执行都会执行方法，无论缓存里是否有值，同时使用新的返回值的替换缓存中的值。这里不同于@Cacheable：@Cacheable如果缓存没有值，从则执行方法并缓存数据，如果缓存有值，则从缓存中获取值

```
    @CachePut(cacheNames="book1", key="#id")
    public Book queryBookCachePut(String id){
        logger.info("queryBookCachePut,id={}",id);
        return repositoryBook.get(id);
    }


```

#### 8.4 @CacheConfig

@CacheConfig: 类级别的注解：如果我们在此注解中定义cacheNames，则此类中的所有方法上 @Cacheable的cacheNames默认都是此值。当然@Cacheable也可以重定义cacheNames的值  

**cacheNames:这个会在redis中出现一个zset集合的数组用来记录已经缓存的数据**

```
@Component
@CacheConfig(cacheNames="booksAll") 
public class BookService2 extends AbstractService {
    private static final Logger logger = LoggerFactory.getLogger(BookService2.class);

    /**
     * 此方法的@Cacheable没有定义cacheNames，则使用类上的注解@CacheConfig里的值 cacheNames
     * @param id
     * @return
     */
    @Cacheable(key="#id")
    public Book queryBookCacheable(String id){
        logger.info("queryBookCacheable,id={}",id);
        return repositoryBook.get(id);
    }

    /**
     * 此方法的@Cacheable有定义cacheNames，则使用此值覆盖类注解@CacheConfig里的值cacheNames
     * 
     * @param id
     * @return
     */
    @Cacheable(cacheNames="books_custom", key="#id")
    public Book queryBookCacheable2(String id){
        logger.info("queryBookCacheable2,id={}",id);
        return repositoryBook.get(id);
    }
}

```

---

#### 注意事项

比如 你的 `cacheNames="books_custom"`  或者 `@CacheConfig(cacheNames="booksAll")`

在redis中会生成一个zset类型的数据  用来存放你所有这个名词下的缓存  比如 key 1  

可以使用 zrange 来查看