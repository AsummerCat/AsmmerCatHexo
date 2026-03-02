---
title: springboot整合es带xpath鉴权
date: 2022-12-15 10:51:41
tags: [java,elasticSearch笔记,SpringBoot,elasticSearch]
---



基于springboot 2.03版本
及其 es 5.5版本
<!--more-->

```

import org.elasticsearch.client.transport.TransportClient;
import org.elasticsearch.common.settings.Settings;
import org.elasticsearch.common.transport.InetSocketTransportAddress;
import org.elasticsearch.xpack.client.PreBuiltXPackTransportClient;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.elasticsearch.core.ElasticsearchTemplate;

import java.net.InetAddress;
import java.net.UnknownHostException;


@Configuration
public class ElasticConfig {

    @Value("${spring.data.elasticsearch.cluster-nodes}")
    private String clusterNodes;
    @Value("${spring.data.elasticsearch.cluster-name}")
    private String clusterName;
    @Value("${spring.data.elasticsearch.username}")
    private String username;
    @Value("${spring.data.elasticsearch.password}")
    private String password;
    @Value("${spring.data.elasticsearch.socketTimeout}")
    private long socketTimeout;


    @Bean
    public TransportClient transportClient() throws UnknownHostException {
        System.out.println("clusterName="+clusterName);
        System.out.println("username="+username);
        System.out.println("password="+password);
        System.out.println("socketTimeout="+socketTimeout);
        System.out.println("clusterNodes="+clusterNodes);

        TransportClient client = new PreBuiltXPackTransportClient(Settings.builder()
                .put("cluster.name", clusterName)
                .put("xpack.security.user", username+":"+password)
                .build());
        try {
            for (String hostPort : clusterNodes.split(",")) {
                String host = hostPort.split(":")[0];
                Integer port = Integer.valueOf(hostPort.split(":")[1]);
                client.addTransportAddress(new InetSocketTransportAddress(InetAddress.getByName(host), port));
            }
        }catch (Exception e){
            e.printStackTrace();
            client.addTransportAddress(new InetSocketTransportAddress(InetAddress.getByName("127.0.0.1"), 9300));
        }
        return client;
    }

    @Bean(name = "elasticsearchTemplate")
    public ElasticsearchTemplate restTemplate() throws Exception {
        return new ElasticsearchTemplate(transportClient());
    }
}

```
```
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.0.3.RELEASE</version>
        <relativePath/> 
    </parent>
    
    
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-elasticsearch</artifactId>
        </dependency>
        <dependency>
            <groupId>org.elasticsearch.client</groupId>
            <artifactId>x-pack-transport</artifactId>
            <version>5.3.0</version>
        </dependency>
```

## 配置文件
```
spring:
  data:
    elasticsearch:
      cluster-nodes: 127.0.0.1:9300,127.0.0.1:9300,127.0.0.1:9300
      repositories:
        enabled: true
      cluster-name: xxxx_es
      username: elastic
      password: xxxx#123
      socketTimeout: 60
```