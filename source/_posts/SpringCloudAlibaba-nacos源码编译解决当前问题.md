---
title: SpringCloudAlibaba-nacos源码编译解决当前问题
date: 2020-04-28 16:25:46
tags: [SpringCloudAlibaba,nacos]
---

# 由于发现个md5解析的问题查看了源码



## 编译

```

单机调试需要加入:
VM options：-Dnacos.standalone=true

执行打包命令
mvn -Prelease-nacos clean install -U  -Dmaven.test.skip=true
```



## 当前版本

2020年4月28日16:27:46

nacos整合sentinel 

```
默认使用的nacos-client是1.14版本的
里面的md5解析 与nacos-config解析的md5 不一致 
初步判断两边的md5算法不一致 

源码中查看了提交记录 发现有人提交了md5算法 并且修改了nacos-config里面的md5算法 其他部分都没有替换

```

## 问题点:

![nacos-client](/img/2020-04-26/nacos-md51.png)

![nacos-config](/img/2020-04-26/nacos-md52.png)

解析的算法不一致

还有部分内容是用`MD5.getInstance().getMD5String(content);`实现的

跟着这个BUG梳理了整个nacos的流程

本来打算提交issue 发现7天前被解决了 但是没有发布到spring-cloud-alibaba-dependencies版本中 

这样会导致你加载的时候可能会导致 无限刷新数据就是匹配不进去

<!--more-->

## 例子:

```
  @Test
    public void testMd5ComparisonMd5Utils(){}{
        //再某些情况下 两个md5算法计算出来的结果不一样
        /**
         * [
         *     {
         *         "resource": "HelloWorld",
         *         "limitApp": "default",
         *         "grade": 1,
         *         "count": 2,
         *         "strategy": 0,
         *         "controlBehavior": 0,
         *         "clusterMode": false
         *     }
         * ]
         */
        StringBuffer sb=new StringBuffer();
        sb.append("[");
        sb.append("    {");
        sb.append("         \"resource\": \"HelloWorld\",");
        sb.append("         \"limitApp\": \"default\",");
        sb.append("         \"grade\": 1,");
        sb.append("        \"count\": 2,");
        sb.append("        \"strategy\": 0,");
        sb.append("        \"controlBehavior\": 0,");
        sb.append("        \"clusterMode\": false");
        sb.append("    }");
        sb.append("]");
        String md5String = MD5.getInstance().getMD5String(sb.toString());
        String md5UtilsString = Md5Utils.getMD5(sb.toString(), Constants.ENCODE);
        Assert.assertEquals("03a499d202d8a04d52481119c462b168", md5String);
        Assert.assertEquals("03a499d202d8a04d52481119c462b168", md5UtilsString);
        Assert.assertEquals(md5String, md5UtilsString);
    }

```



## 使用版本

```java
        <dependencies>
            <dependency>
                <groupId>com.alibaba.cloud</groupId>
                <artifactId>spring-cloud-alibaba-dependencies</artifactId>
                <version>2.2.0.RELEASE</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
```

解决办法

```
1.手动拉去 github上最新的nacos包 
2.或者减低nacos版本 避免两边算法不一致
```

```
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
            <exclusions>
                <exclusion>
                    <groupId>com.alibaba.nacos</groupId>
                    <artifactId>nacos-client</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        
    <dependency>
            <groupId>com.alibaba.nacos</groupId>
            <artifactId>nacos-client</artifactId>
            <version>1.2.1</version>
        </dependency>
```

移除内置的nacos-client版本 手动添加