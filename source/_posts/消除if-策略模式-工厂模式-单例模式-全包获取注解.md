---
title: 消除if-策略模式+工厂模式+单例模式-全包获取注解
date: 2019-08-20 20:05:29
tags: [java,设计模式]
---

# 消除if-策略模式+工厂模式+单例模式-单路径获取注解

这样处理的好处是 : 如果有多个if 判断执行多个分支 后期修改代码比较麻烦

使用注解 标记这个分支 使用工厂 自动匹配这个分支执行

以后就只要维护注解 和具体分支就可以了 不用改动其他的 

这个跟单路径的差不多 不过这边就是可用获取整个包下所有标记注解的分支

# 创建注解

```java
/**
 * 首页指标注解
 *
 * @author cxc
 * @date 2019年8月20日11:25:10
 */
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface IndexBi {
    /**
     * 指标名称
     *
     * @return
     */
    String value();
}
```

<!--more-->

# 创建策略接口及其实现

## 定义首页bi接口

```java
/**
 * 首页bi接口
 *
 * @author cxc
 * @date 2019年8月20日10:28:58
 */
public interface BiService {

    /**
     * 获取数据
     *
     * @return
     */
    public List<Map<String,Object>> getData(IndexBiEntity indexBi);
}

```

## 创建实现类 及其标记注解

```java
/**
 * 首页BI 
 *
 * @author cxc
 * @date 2019年8月20日13:41:27
 */
@IndexBi("antimicrobialUseRate")
@Service
public class BiAntimicrobialUseRateServiceImpl implements BiService {
    @Override
    public List<Map<String, Object>> getData(IndexBiEntity indexBi) {
        return null;
    }
}
```

```java
/**
 * 首页BI
 *
 * @author cxc
 * @date 2019年8月20日13:41:27
 */
@IndexBi("DDDS")
@Service
public class BiDDDSServiceImpl implements BiService {
    @Override
    public List<Map<String, Object>> getData(IndexBiEntity indexBi) {
        return null;
    }
}
```

有需要的话继续扩展，创建实现类即可

# 创建工厂类 

- 工厂类这边的话就是 交给spring 处理生成单例的bean
- 然后创建一个spring context listener 获取被spring捕获的注解 set到工厂类的静态变量中

```java
/**
 * 工厂模式 根据查看的指标的类型自动加载
 *
 * @author cxc
 * @date 2019年8月8日17:50:38
 */
@Component
public class IndexBiFactory {
    /**
     * 维护的类型和className
     */
    private static HashMap<String, String> IndexBiMap = new HashMap<>();


    /**
     * 根据channelId获取对应的具体实现
     *
     * @return
     */
    public BiService create(String type) throws ClassNotFoundException, IllegalAccessException, InstantiationException {
        //1.获取渠道对应的类名
        String clazz = IndexBiMap.get(type);
        if (StringUtils.isEmpty(clazz)) {
            throw new BusinessException("未定义的首页BI类型,无法转换实体");
        }
        Class clazz_ = Class.forName(clazz);
        /**
         * newInstance ：工厂模式经常使用newInstance来创建对象，newInstance()是实现IOC、反射、依赖倒置 等技术方法的必然选择
         *      调用class的加载方法加载某个类，然后实例化
         *
         * new 只能实现具体类的实例化，不适合于接口编程
         */
        return (BiService) clazz_.newInstance();
    }


    public HashMap<String, String> getIndexBiMap() {
        return IndexBiMap;
    }

    public void setIndexBiMap(HashMap<String, String> indexBiMap) {
        IndexBiMap = indexBiMap;
    }
}

```

# 这边就了 获取项目下所有注解的部分了

```java
/**
 * Spring启动后获取所有拥有特定注解的Bean
 *
 * @author cxc
 * @date 2019年8月20日11:48:58
 */
@Component
public class ContextRefreshedListener implements ApplicationListener<ContextRefreshedEvent> {
    @Autowired
    private IndexBiFactory biFactory;

    @Override
    public void onApplicationEvent(ContextRefreshedEvent event) {
        HashMap<String, String> indexBiMap = new HashMap<>();
        // 根容器为Spring容器
        if (event.getApplicationContext().getParent() == null) {
            Map<String, Object> beans = event.getApplicationContext().getBeansWithAnnotation(IndexBi.class);
            for (Object bean : beans.values()) {
                bean.getClass().getAnnotation(IndexBi.class).value();
                indexBiMap.put(bean.getClass().getAnnotation(IndexBi.class).value(), bean.getClass().getName());
            }
            biFactory.setIndexBiMap(indexBiMap);
        }
    }
}
```

获取到bean后 注入到Factory的map中

# 创建上下文

```java
/**
 * 首页BI指标上下文
 * @author cxc
 * @date 2019年8月20日14:45:10
 */
@Component
public class IndexBiContext {
    @Autowired
    private IndexBiFactory indexBiFactory;

    /**
     * 执行目标方法 获取数据
     */
    public List<Map<String,Object>> getData(IndexBiEntity indexBi) throws Exception {
        //2.调用具体实现
        BiService biService = indexBiFactory.create(indexBi.getBiType());
        //3.执行并返回结果
        return biService.getData(indexBi);
    }
}
```

# 执行方法

```java
@RestController
public class testController {
    private static final Logger logger = LoggerFactory.getLogger(testController.class);

    @Autowired
    private IndexBiContext indexBiContext;

    @RequestMapping("testAA")
    public List<Map<String, Object>> getIndexBi(IndexBiEntity indexBi) {
        try {
            return indexBiContext.getData(indexBi);
        } catch (Exception e) {
            logger.error("獲取數據失敗", e);
            return null;
        }
    }
}
```

# 实体

```java
/**
 * 首页Bi搜索条件
 *
 * @author cxc
 * @date 2019年8月20日14:18:34
 */
public class IndexBiEntity {

    private String departId;

    private String startTime;

    private String endTime;

    private String biType;


    public String getDepartId() {
        return departId;
    }

    public void setDepartId(String departId) {
        this.departId = departId;
    }

    public String getStartTime() {
        return startTime;
    }

    public void setStartTime(String startTime) {
        this.startTime = startTime;
    }

    public String getEndTime() {
        return endTime;
    }

    public void setEndTime(String endTime) {
        this.endTime = endTime;
    }

    public String getBiType() {
        return biType;
    }

    public void setBiType(String biType) {
        this.biType = biType;
    }
}

```



