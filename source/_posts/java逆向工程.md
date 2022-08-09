---
title: java逆向工程
date: 2022-08-09 15:26:50
tags: [java,mybatis,mybatis-plus-generator]
---
# java逆向工程

## 导入pom
```
        <!--generator start-->
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-generator</artifactId>
            <version>3.4.1</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.apache.velocity</groupId>
            <artifactId>velocity-engine-core</artifactId>
            <version>2.0</version>
            <scope>test</scope>
        </dependency>
        <!--generator end-->
```
<!--more-->

## 编写构造类
```
package com.xxx;

import com.baomidou.mybatisplus.annotation.DbType;
import com.baomidou.mybatisplus.annotation.IdType;
import com.baomidou.mybatisplus.core.toolkit.StringPool;
import com.baomidou.mybatisplus.generator.AutoGenerator;
import com.baomidou.mybatisplus.generator.InjectionConfig;
import com.baomidou.mybatisplus.generator.config.*;
import com.baomidou.mybatisplus.generator.config.po.TableInfo;
import com.baomidou.mybatisplus.generator.config.rules.DateType;
import com.baomidou.mybatisplus.generator.config.rules.NamingStrategy;

import java.util.ArrayList;
import java.util.List;

public class MyBatisPlusGenerator {

    public static void main(String[] args) {

        String path = "D:\\work\\xxxx\\src\\main";

        //1. 全局配置
        GlobalConfig config = new GlobalConfig();
        // 是否支持AR模式
        config.setActiveRecord(true)
                // 作者
                .setAuthor("xxx")
                // 生成路径，最好使用绝对路径
                .setOutputDir(path+"\\java")
                // 文件覆盖
                .setFileOverride(true)
                // 主键策略
                .setIdType(IdType.AUTO)

                .setDateType(DateType.ONLY_DATE)
                // 设置生成的service接口的名字的首字母是否为I，默认Service是以I开头的
                .setServiceName("%sService")

                //实体类结尾名称
//                .setEntityName("%s")

                //生成基本的resultMap
                .setBaseResultMap(true)

                //不使用AR模式
                .setActiveRecord(false)

                //生成基本的SQL片段
                .setBaseColumnList(true)
                // 设置日期类型
                .setDateType(DateType.ONLY_DATE)
                .setSwagger2(true);

        //2. 数据源配置
        DataSourceConfig dsConfig = new DataSourceConfig();
        // 设置数据库类型
        dsConfig.setDbType(DbType.MYSQL)
                .setDriverName("com.mysql.cj.jdbc.Driver")
                .setUrl("")
                .setUsername("")
                .setPassword("");

        //3. 策略配置globalConfiguration中
        StrategyConfig stConfig = new StrategyConfig();

        //全局大写命名
        stConfig.setCapitalMode(true)
                // 数据库表映射到实体的命名策略
                .setNaming(NamingStrategy.underline_to_camel)

                //使用lombok
                .setEntityLombokModel(true)

                //使用restcontroller注解
                .setRestControllerStyle(true)

                // 生成的表, 支持多表一起生成，以数组形式填写
//                .setInclude("product", "banner", "address", "coupon", "product_order");
                .setInclude("agency_tj_module");
//                .setInclude(scanner("表名，多个英文逗号分割").split(","));

        //4. 包名策略配置
        PackageConfig pkConfig = new PackageConfig();
        pkConfig.setParent("com.xxx.fuzhou.operation_center")
                .setMapper("mapper")
                .setService("service")
                .setController("controller")
                .setEntity("entity");

        // 自定义配置
        InjectionConfig cfg = new InjectionConfig() {
            @Override
            public void initMap() {
                // to do nothing
            }
        };

// 如果模板引擎是 freemarker
// String templatePath = "/templates/mapper.xml.ftl";
// 如果模板引擎是 velocity
        String templatePath = "/templates/mapper.xml.vm";

// 自定义输出配置
        List<FileOutConfig> focList = new ArrayList<>();
// 自定义配置会被优先输出
// 这里自定义配置的是*Mapper.xml文件
// 所以templatePath = "/templates/mapper.xml.vm";
// 如果你想自定义配置其它 修改templatePath即可
        focList.add(new FileOutConfig(templatePath) {
            @Override
            public String outputFile(TableInfo tableInfo) {
                // 自定义输出文件名 如果你 Entity 设置了前后缀
//                String entityName = tableInfo.getEntityName();
//                int length = entityName.length();
//                entityName = entityName.substring(0, length - 6);
                return path + "/resources/mybatis/mapper/" +
                        tableInfo.getEntityName() + "Mapper" + StringPool.DOT_XML;
            }
        });

        cfg.setFileOutConfigList(focList);

        TemplateConfig templateConfig=new TemplateConfig();
        templateConfig.setXml(null);



        //5. 整合配置
        AutoGenerator ag = new AutoGenerator();
        ag.setGlobalConfig(config)
                .setDataSource(dsConfig)
                .setStrategy(stConfig)
                .setPackageInfo(pkConfig)
                .setCfg(cfg)
                .setTemplate(templateConfig);

        //6. 执行操作
        ag.execute();
        System.out.println("======= 相关代码生成完毕  ========");
    }

}

```