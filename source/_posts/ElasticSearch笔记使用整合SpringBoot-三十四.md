---
title: ElasticSearch笔记使用整合SpringBoot(三十四)
date: 2020-09-01 23:22:06
tags: [ElasticSearch笔记]
---

# [demo地址](https://github.com/AsummerCat/es_api_demo)

# client集群的自动探查

我们一般手动配置节点
但是如果成千上百的节点如何处理,这就需要`集群自动探查`了  

es client提供了一种集群节点自动探查的功能,打开这个自动探查机制以后,es client会根据我们手动那个指定的几个节点连接过去,然后通过集群状态自动获取当前集群的所有`data node`,然后用这个完整的列表更新我们内部要发哦上请求的`node list`   
默认每隔5秒钟,就会更新一次`node list`  

注意: es client 是不会将master node纳入node list的,
因为要避免给master node 发送搜索等请求.  
使用自动探查的话,我们就只要指定几个master node,就好了,client会自动去探查集群的所有节点,并且自动刷新.  
配置类写法:
```
@Data
@Configuration
@ConfigurationProperties(prefix = "elastic")
public class ElasticsearchConfig {

	private String host;
	private Long connectTimeout;
	private Long socketTimeout;
	//private int port;


	@Bean
	public RestHighLevelClient client() {
		InetSocketAddress inetSocketAddress1 = new InetSocketAddress("127.0.0.1", 9200);
		InetSocketAddress inetSocketAddress2 = new InetSocketAddress("127.0.0.2", 9200);
		InetSocketAddress inetSocketAddress3 = new InetSocketAddress("127.0.0.3", 9200);
		InetSocketAddress inetSocketAddress4 = new InetSocketAddress("127.0.0.4", 9200);

		ClientConfiguration clientConfiguration = ClientConfiguration.builder()
				.connectedTo(inetSocketAddress1,inetSocketAddress2,inetSocketAddress3,inetSocketAddress4)
				.withConnectTimeout(connectTimeout)
				.withSocketTimeout(socketTimeout)
				.build();
		return RestClients.create(clientConfiguration).rest();
	}
}

```

# JAVA配置类版本
yml文件
```
elastic:
  host: 127.0.0.1:9200
  connectTimeout: 10
  socketTimeout: 5

```

配置类
```
@Data
@Configuration
@ConfigurationProperties(prefix = "elastic")
public class ElasticsearchConfig {

    private String host;
    private Long connectTimeout;
    private Long socketTimeout;
    //private int port;


    @Bean
    public RestHighLevelClient client() {
        ClientConfiguration clientConfiguration = ClientConfiguration.builder()
                .connectedTo(host)
                .withConnectTimeout(connectTimeout)
                .withSocketTimeout(socketTimeout)
                .build();
        return RestClients.create(clientConfiguration).rest();
    }
}


```



# JAVA配置文件版本
springboot2.3版本以上
### 修改yml
```
spring:
  application:
    name: es_api_demo
  data:
    elasticsearch:
      client:
        reactive:   # 高级连接器
          endpoints: 127.0.0.1:9200
          connection-timeout: 10  #链接到es的超时时间，毫秒为单位，默认10秒（10000毫秒）
          socket-timeout: 5        #读取和写入的超时时间，单位为毫秒，默认5秒（5000毫秒）
  elasticsearch:
    rest:    # 低级连接器
      uris: 127.0.0.1:9200

```
这样就可以直接用了

# 使用方式
1.创建ElasticsearchRepository的子类使用
```
@Repository
public interface PersonRepository extends ElasticsearchRepository<Person, String> {

	List<Person> findByName(String keyWord);
	
	//其中这边里的跟JPA的内容一样
	
	默认自带了save insert等方法
	需要实体上标记了es的节点
}


```
标记es的实体
```
@Document(indexName = "user")
@Data
public class Person {

    /**
     * 必须有id注解
     */
    @Id
    private String id;
    private String name;
    private int age;
    private String idCard;


}

```

2.使用原生client语法
```
	@Autowired
	private RestHighLevelClient restClient;

```