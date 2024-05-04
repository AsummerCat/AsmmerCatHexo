---
title: 动态加载多数据源Spring工厂Bean
date: 2024-05-04 15:48:34
tags: [多数据源, Spring]
---
# 实现动态加载多个数据源

## 配置文件

```
datasource.datasource[0].driverClassName=dm.jdbc.driver.DmDriver
datasource.datasource[0].url=jdbc:dm://192.168.0.1:1111
datasource.datasource[0].username=1
datasource.datasource[0].password=1
datasource.datasource[0].type=1
datasource.datasource[0].dbName=1

datasource.datasource[1].driverClassName=dm.jdbc.driver.DmDriver
datasource.datasource[1].url=jdbc:dm://192.168.0.1:1111
datasource.datasource[1].username=1
datasource.datasource[1].password=1
datasource.datasource[1].type=1
datasource.datasource[1].dbName=1

```

<!--more-->

### 配置文件实体

    import com.linjingc.TableInfoData;
    import com.linjingc.DataSourceConfig;
    import org.apache.commons.dbcp.BasicDataSource;
    import org.springframework.boot.context.properties.ConfigurationProperties;
    import org.springframework.context.annotation.Configuration;

    import javax.annotation.PostConstruct;
    import javax.sql.DataSource;
    import java.util.HashMap;
    import java.util.List;


    @Configuration
    @ConfigurationProperties(prefix = "datasource")
    public class DbConfig {
        public static HashMap<String, DataSource> dataSourceMap = new HashMap<>();

        private List<DbInfo> datasource;


        public List<DbInfo> getDatasource() {
            return datasource;
        }

        public void setDatasource(List<DbInfo> datasource) {
            this.datasource = datasource;
        }


        @PostConstruct
        public void createDataSource() {
            List<DbInfo> datasource = this.getDatasource();
            for (DbInfo dbInfo : datasource) {
                DataSource dataSource = buildDataSource(dbInfo);
                dataSourceMap.put(dbInfo.getType(), dataSource);
            }
        }


        private DataSource buildDataSource(DbInfo dbInfo) {
            BasicDataSource dataSource = new BasicDataSource();
            dataSource.setDriverClassName(dbInfo.getDriverClassName());
            dataSource.setUrl(dbInfo.getUrl());
            dataSource.setUsername(dbInfo.getUsername());
            dataSource.setPassword(dbInfo.getPassword());
            //初始化的连接数
            dataSource.setInitialSize(10);
            //最大连接数量
            dataSource.setMaxActive(300);
            //最大空闲数
            dataSource.setMaxIdle(50);
            //最小空闲数
            dataSource.setMinIdle(1);
            //最大等待时间
            dataSource.setMaxWait(60000);
            dataSource.setValidationQuery("select 1 ");
            dataSource.setTestOnBorrow(true);
            dataSource.setDefaultAutoCommit(true);
            return dataSource;
        }
    }



    import lombok.AllArgsConstructor;
    import lombok.Data;
    import lombok.NoArgsConstructor;

    @Data
    @AllArgsConstructor
    @NoArgsConstructor
    public class DbInfo {
        /**
         * 数据库驱动
         */
        private String driverClassName;
        /**
         * 数据库连接
         */
        private String url;
        /**
         * 数据库用户名
         */
        private String username;
        /**
         *  数据库密码
         */
        private String password;
        /**
         * 数据库标记
         */
        private String type;
        /**
         * 数据库名称
         */
        private String dbName;
    }

### 构建工厂Bean

    @Component
    @Data
    public class DbFactoryBean implements FactoryBean<DataSource> {
        @Setter
        private static String dbType;

        /**
         * 返回创建的对象
         */
        @Override
        public DataSource getObject() {
            System.out.println("当前使用库:"+dbType);
            return MapUtil.get(DbConfig.dataSourceMap, dbType, DataSource.class);
        }

        /**
         * 返回创建的对象的类型
         */
        @Override
        public Class<?> getObjectType() {
            return DataSource.class;
        }

        /**
         * 是单例? true:是 false:不是
         */
        @Override
        public boolean isSingleton() {
            return false;
        }
    }

### 工具类

    public class DbCheckUtils {

        public static DataSource changeDb(String type) {
            DbFactoryBean.setDbType(type);
            return SpringUtil.getBean("dbFactoryBean");
        }

    }


    使用:
    DbCheckUtils.changeDb(dbName);

