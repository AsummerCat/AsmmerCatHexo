---
title: SpringCloudAlibaba-nacos整合sentinel后的工作流程
date: 2020-04-27 17:21:03
tags: [SpringCloudAlibaba,sentinel,Nacos]
---

# nacos整合sentinel后的加载流程

## 导入pom

```java
 <dependency>
            <groupId>com.alibaba.csp</groupId>
            <artifactId>sentinel-datasource-nacos</artifactId>
        </dependency>
```

## 流程

### NacosDataSource调用初始化方法

类名:com.alibaba.csp.sentinel.datasource.nacos.NacosDataSource

```java
initNacosListener();
//这里一步是 根据上面的初始化获取到数据后 加载进环境里面的配置 也就是初始化配置信息的步骤 跟后面的定时线程分开
loadInitialConfig(); 
//常规情况这边不修改的话 就是nacos中的数据了 修改了数据被clientWorker10s刷新到 然后重新赋值
```

### 获取nacos地址 NacosFactory.createConfigService(this.properties);

<!--more-->

### 添加监听器

```java
configService.addListener(dataId, groupId, configListener);
这里就是你的dataid 和分组 加监听器
```

底层->调用worker.addListener(String dataId, String group, Listener listener)

在com.alibaba.nacos.client.config;包下NacosConfigService 私有参数:NacosConfigService

去判断缓存池中是否存在dataid和group组合的key 不存在的话创建一个缓存(CacheData) 添加一个this.isInitializing = true;标识

然后循环监听器 给当前CacheData加入监听器

```java
for (Listener listener : listeners) {
            cache.addListener(listener);
        }
```

### 然后 com.alibaba.cloud.nacos.refresh; 的 NacosContextRefresher初始化监听器

这个是监听springboot的

```
@Override
	public void onApplicationEvent(ApplicationReadyEvent event) {
		// many Spring context
		if (this.ready.compareAndSet(false, true)) {
			this.registerNacosListenersForApplications();
		}
	}
```

这边就开始初始化项目名称的dataId和group 获取

获取完毕 开始启动项目

### 启动中会加载NacosConfigBootstrapConfiguration

```
其中 会判断 然后加载

	@Bean
	@ConditionalOnMissingBean
	public NacosConfigManager nacosConfigManager(
			NacosConfigProperties nacosConfigProperties) {
		return new NacosConfigManager(nacosConfigProperties);
	}
```

`com.alibaba.cloud.nacos.NacosConfigManager` 在config包中

在调用new NacosConfigManager(nacosConfigProperties)构造方法中

会调用`createConfigService(nacosConfigProperties);`

```
com.alibaba.nacos.client.config.NacosConfigService
利用反射加代理 创建出这个内容
调用 NacosConfigService(Properties properties);
里面会创建一个客户端进程 worker = new ClientWorker(agent, configFilterChainManager, properties);
这个东西里面会开启一个异步线程 也就是我们所说的会去定时刷新config配置的地方
拉取nacos请求 啥的
```

### ClientWorker初始化   工作线程

这个com.alibaba.nacos.client.config.impl  在naocs-clientbao包中

```
  public ClientWorker(final HttpAgent agent, final ConfigFilterChainManager configFilterChainManager, final Properties properties) {
        this.agent = agent;
        this.configFilterChainManager = configFilterChainManager;

        // Initialize the timeout parameter

        init(properties);

        executor = Executors.newScheduledThreadPool(1, new ThreadFactory() {
            @Override
            public Thread newThread(Runnable r) {
                Thread t = new Thread(r);
                t.setName("com.alibaba.nacos.client.Worker." + agent.getName());
                t.setDaemon(true);
                return t;
            }
        });

        executorService = Executors.newScheduledThreadPool(Runtime.getRuntime().availableProcessors(), new ThreadFactory() {
            @Override
            public Thread newThread(Runnable r) {
                Thread t = new Thread(r);
                t.setName("com.alibaba.nacos.client.Worker.longPolling." + agent.getName());
                t.setDaemon(true);
                return t;
            }
        });

        executor.scheduleWithFixedDelay(new Runnable() {
            @Override
            public void run() {
                try {
                    checkConfigInfo();
                } catch (Throwable e) {
                    LOGGER.error("[" + agent.getName() + "] [sub-check] rotate check error", e);
                }
            }
        }, 1L, 10L, TimeUnit.MILLISECONDS);
    }
```

创建一个守护线程  并且是1L开始 每隔10秒进行请求一次checkConfigInfo();

```
 public void checkConfigInfo() {
        // 分任务 从cacheMap 获取到我们需要承担cache 分任务进行加载
        int listenerSize = cacheMap.get().size();
        // 向上取整为批数
        int longingTaskCount = (int) Math.ceil(listenerSize / ParamUtil.getPerTaskConfigSize());
        if (longingTaskCount > currentLongingTaskCount) {
            for (int i = (int) currentLongingTaskCount; i < longingTaskCount; i++) {
                // 要判断任务是否在执行 这块需要好好想想。 任务列表现在是无序的。变化过程可能有问题
                executorService.execute(new LongPollingRunnable(i));
            }
            currentLongingTaskCount = longingTaskCount;
        }
    }
```



### 在启动过程中会开启一个线程 clientWorker进行监听dataid是否更新 刷入缓存池

```
clientWorker的内部类   new LongPollingRunnable(i)->public void run() {)
会执行
```

#### 第一步  首先判断本地是否有本地缓存数据 如果是使用本地缓存数据 继续判断Md5是否相等

```
  // check failover config
                for (CacheData cacheData : cacheMap.get().values()) {
                    if (cacheData.getTaskId() == taskId) {
                        cacheDatas.add(cacheData);
                        try {
                            checkLocalConfig(cacheData);
                            if (cacheData.isUseLocalConfigInfo()) {
                                cacheData.checkListenerMd5();
                            }
                        } catch (Exception e) {
                            LOGGER.error("get local config info error", e);
                        }
                    }
                }
```

#### 第二步 从Server获取值变化了的DataID列表. 然后进行一系列请求加载处理刷入缓存

```java
这个步骤比较深 -> 会发送请求给nacos获取数据 再进行判断处理 改变cache.isInitializing状态
List<String> changedGroupKeys = checkUpdateDataIds(cacheDatas, inInitializingCacheList);

```

##### 1.checkUpdateDataIds() 如何执行

```
List<String> checkUpdateDataIds(List<CacheData> cacheDatas, List<String> inInitializingCacheList) throws IOException {
        StringBuilder sb = new StringBuilder();
        --遍历缓存的key
        for (CacheData cacheData : cacheDatas) {
           --判断本地缓存是否存在 不存在继续下一步调用
            if (!cacheData.isUseLocalConfigInfo()) {
                sb.append(cacheData.dataId).append(WORD_SEPARATOR);
                sb.append(cacheData.group).append(WORD_SEPARATOR);
                if (StringUtils.isBlank(cacheData.tenant)) {
                    sb.append(cacheData.getMd5()).append(LINE_SEPARATOR);
                } else {
                    sb.append(cacheData.getMd5()).append(WORD_SEPARATOR);
                    sb.append(cacheData.getTenant()).append(LINE_SEPARATOR);
                }
                //判断是否在初始化过程中  
                if (cacheData.isInitializing()) {
                    // cacheData 首次出现在cacheMap中&首次check更新
                    是的话加入这个集合 
                    inInitializingCacheList
                        .add(GroupKey.getKeyTenant(cacheData.dataId, cacheData.group, cacheData.tenant));
                }
            }
        }
        //判断集合中是否存在数据
        boolean isInitializingCacheList = !inInitializingCacheList.isEmpty();
        //检查并更新config中的data数据 ->更新的数据是未初始化的数据isInitializingCacheList
        return checkUpdateConfigStr(sb.toString(), isInitializingCacheList);
    }
```

##### 2.checkUpdateConfigStr()如何执行

```
 /**
     * 从Server获取值变化了的DataID列表。返回的对象里只有dataId和group是有效的。 保证不返回NULL。
     */
    List<String> checkUpdateConfigStr(String probeUpdateString, boolean isInitializingCacheList) throws IOException {
 
        List<String> params = Arrays.asList(Constants.PROBE_MODIFY_REQUEST, probeUpdateString);

        List<String> headers = new ArrayList<String>(2);
        headers.add("Long-Pulling-Timeout");
        headers.add("" + timeout);
       
        
        //告诉服务器不要挂起我如果新的初始化cacheData加入 也就是如果是新初始化的 就加入一个头信息
        if (isInitializingCacheList) {
            headers.add("Long-Pulling-Timeout-No-Hangup");
            headers.add("true");
        }
        
        //判断cacheData的key是否为空
        if (StringUtils.isBlank(probeUpdateString)) {
            return Collections.emptyList();
        }

        try {
        //向nacos发送请求key 
            HttpResult result = agent.httpPost(Constants.CONFIG_CONTROLLER_PATH + "/listener", headers, params,
                agent.getEncode(), timeout);
            //请求成功 并且保持心跳
            if (HttpURLConnection.HTTP_OK == result.code) {
                setHealthServer(true);
                //从响应中获取得到变化groupkey
                return parseUpdateDataIdResponse(result.content);
            } else {
            //请求失败 心跳返回false 然后返回空集合
                setHealthServer(false);
                LOGGER.error("[{}] [check-update] get changed dataId error, code: {}", agent.getName(), result.code);
            }
        } catch (IOException e) {
            setHealthServer(false);
            LOGGER.error("[" + agent.getName() + "] [check-update] get changed dataId exception", e);
            throw e;
        }
        return Collections.emptyList();
    }
```

##### 3.parseUpdateDataIdResponse()如何执行

```
private List<String> parseUpdateDataIdResponse(String response) {
     //判断返回结果是否为null 为null 直接返回
        if (StringUtils.isBlank(response)) {
            return Collections.emptyList();
        }

        try {
        //获取到被修改过的 nacos传回来的数据
            response = URLDecoder.decode(response, "UTF-8");
        } catch (Exception e) {
            LOGGER.error("[" + agent.getName() + "] [polling-resp] decode modifiedDataIdsString error", e);
        }

        List<String> updateList = new LinkedList<String>();
        
        //进行切割处理 并且存入updateList;
        for (String dataIdAndGroup : response.split(LINE_SEPARATOR)) {
            if (!StringUtils.isBlank(dataIdAndGroup)) {
                String[] keyArr = dataIdAndGroup.split(WORD_SEPARATOR);
                String dataId = keyArr[0];
                String group = keyArr[1];
                if (keyArr.length == 2) {
                    updateList.add(GroupKey.getKey(dataId, group));
                    LOGGER.info("[{}] [polling-resp] config changed. dataId={}, group={}", agent.getName(), dataId, group);
                } else if (keyArr.length == 3) {
                    String tenant = keyArr[2];
                    updateList.add(GroupKey.getKeyTenant(dataId, group, tenant));
                    LOGGER.info("[{}] [polling-resp] config changed. dataId={}, group={}, tenant={}", agent.getName(),
                        dataId, group, tenant);
                } else {
                    LOGGER.error("[{}] [polling-resp] invalid dataIdAndGroup error {}", agent.getName(), dataIdAndGroup);
                }
            }
        }
        return updateList;
    }
```

##### 4.利用上面返回的更新后的结果集  下一步就是更新cacheMap中的内容

根据上面checkUpdateDataIds()的返回值进行判断是否更新了数据 如果有更新执行以下循环赋值给cacheMap

```
for (String groupKey : changedGroupKeys) {
                    String[] key = GroupKey.parseKey(groupKey);
                    String dataId = key[0];
                    String group = key[1];
                    String tenant = null;
                    if (key.length == 3) {
                        tenant = key[2];
                    }
                    try {
                    //获取内容 这里的话会再去请求一次nacos获取当前key的最新数据
                        String content = getServerConfig(dataId, group, tenant, 3000L);
                       //获取原先key 
                        CacheData cache = cacheMap.get().get(GroupKey.getKeyTenant(dataId, group, tenant));
                        //赋值内容
                        cache.setContent(content);
                        //写入日志
                        LOGGER.info("[{}] [data-received] dataId={}, group={}, tenant={}, md5={}, content={}",
                            agent.getName(), dataId, group, tenant, cache.getMd5(),
                            ContentUtils.truncateContent(content));
                    } catch (NacosException ioe) {
                        String message = String.format(
                            "[%s] [get-update] get changed config exception. dataId=%s, group=%s, tenant=%s",
                            agent.getName(), dataId, group, tenant);
                        LOGGER.error(message, ioe);
                    }
                }
```

##### 5.最后一步 就是判断当前key是否初始化了 如果初始化了判断MD5并且修改初始化状态

```java
   for (CacheData cacheData : cacheDatas) {
                   //判断是否处于初始化状态 , 或者当前数据在返回的更新结果集中 修改初始化状态
                    if (!cacheData.isInitializing() || inInitializingCacheList
                        .contains(GroupKey.getKeyTenant(cacheData.dataId, cacheData.group, cacheData.tenant))) {
                        //进行判断MD5是否相等 不相等进行安全回调给listen的所有者通知md5已经更新了(数据更新)
                        cacheData.checkListenerMd5();
                        cacheData.setInitializing(false);
                    }
                }
                //清理list结果
               inInitializingCacheList.clear();
                //再次进行线程循环
                executorService.execute(this);      
```

##### cacheData.checkListenerMd5();

```
 private void safeNotifyListener(final String dataId, final String group, final String content,
                                    final String md5, final ManagerListenerWrap listenerWrap) {
       //监听器所有者
       final Listener listener = listenerWrap.listener;
        创建一个Runnable
        Runnable job = new Runnable() {
            @Override
            public void run() {
                ClassLoader myClassLoader = Thread.currentThread().getContextClassLoader();
                ClassLoader appClassLoader = listener.getClass().getClassLoader();
                try {
                    if (listener instanceof AbstractSharedListener) {
                        AbstractSharedListener adapter = (AbstractSharedListener) listener;
                        adapter.fillContext(dataId, group);
                        LOGGER.info("[{}] [notify-context] dataId={}, group={}, md5={}", name, dataId, group, md5);
                    }
                    // 执行回调之前先将线程classloader设置为具体webapp的classloader，以免回调方法中调用spi接口是出现异常或错用（多应用部署才会有该问题）。
                    Thread.currentThread().setContextClassLoader(appClassLoader);

                    ConfigResponse cr = new ConfigResponse();
                    cr.setDataId(dataId);
                    cr.setGroup(group);
                    cr.setContent(content);
                    configFilterChainManager.doFilter(null, cr);
                    String contentTmp = cr.getContent();
                    listener.receiveConfigInfo(contentTmp);
                    listenerWrap.lastCallMd5 = md5;
                    LOGGER.info("[{}] [notify-ok] dataId={}, group={}, md5={}, listener={} ", name, dataId, group, md5,
                        listener);
                } catch (NacosException de) {
                    LOGGER.error("[{}] [notify-error] dataId={}, group={}, md5={}, listener={} errCode={} errMsg={}", name,
                        dataId, group, md5, listener, de.getErrCode(), de.getErrMsg());
                } catch (Throwable t) {
                    LOGGER.error("[{}] [notify-error] dataId={}, group={}, md5={}, listener={} tx={}", name, dataId, group,
                        md5, listener, t.getCause());
                } finally {
                    Thread.currentThread().setContextClassLoader(myClassLoader);
                }
            }
        };

        final long startNotify = System.currentTimeMillis();
        try {
            if (null != listener.getExecutor()) {
                listener.getExecutor().execute(job);
            } else {
                job.run();
            }
        } catch (Throwable t) {
            LOGGER.error("[{}] [notify-error] dataId={}, group={}, md5={}, listener={} throwable={}", name, dataId, group,
                md5, listener, t.getCause());
        }
        final long finishNotify = System.currentTimeMillis();
        LOGGER.info("[{}] [notify-listener] time cost={}ms in ClientWorker, dataId={}, group={}, md5={}, listener={} ",
            name, (finishNotify - startNotify), dataId, group, md5, listener);
    }
```

