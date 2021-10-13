---
title: java生成数据库设计
date: 2021-10-13 22:46:58
tags: [java]
---

# java生成数据库设计
github:https://github.com/pingfangushi/screw

## 方式一: 写代码
```
导入pom.xml
	<dependency>
			<groupId>com.zaxxer</groupId>
			<artifactId>HikariCP</artifactId>
			<version>3.4.5</version>
		</dependency>
		<dependency>
			<groupId>cn.smallbun.screw</groupId>
			<artifactId>screw-core</artifactId>
			<version>1.0.5</version>
		</dependency>
```

<!--more-->
### 文件类型:
```
EngineFileType.HTML 
支持三种
  HTML(".html", "documentation_html", "HTML文件"),
    WORD(".doc", "documentation_word", "WORD文件"),
    MD(".md", "documentation_md", "Markdown文件");
```

```
/**
 * 文档生成
 */
void documentGeneration() {
   //数据源
   HikariConfig hikariConfig = new HikariConfig();
   hikariConfig.setDriverClassName("com.mysql.cj.jdbc.Driver");
   hikariConfig.setJdbcUrl("jdbc:mysql://127.0.0.1:3306/database");
   hikariConfig.setUsername("root");
   hikariConfig.setPassword("password");
   //设置可以获取tables remarks信息
   hikariConfig.addDataSourceProperty("useInformationSchema", "true");
   hikariConfig.setMinimumIdle(2);
   hikariConfig.setMaximumPoolSize(5);
   DataSource dataSource = new HikariDataSource(hikariConfig);
   //生成配置
   EngineConfig engineConfig = EngineConfig.builder()
         //生成文件路径
         .fileOutputDir(fileOutputDir)
         //打开目录
         .openOutputDir(true)
         //文件类型
         .fileType(EngineFileType.HTML)
         //生成模板实现
         .produceType(EngineTemplateType.freemarker)
         //自定义文件名称
         .fileName("自定义文件名称").build();

   //忽略表
   ArrayList<String> ignoreTableName = new ArrayList<>();
   ignoreTableName.add("test_user");
   ignoreTableName.add("test_group");
   //忽略表前缀
   ArrayList<String> ignorePrefix = new ArrayList<>();
   ignorePrefix.add("test_");
   //忽略表后缀    
   ArrayList<String> ignoreSuffix = new ArrayList<>();
   ignoreSuffix.add("_test");
   ProcessConfig processConfig = ProcessConfig.builder()
         //指定生成逻辑、当存在指定表、指定表前缀、指定表后缀时，将生成指定表，其余表不生成、并跳过忽略表配置	
		 //根据名称指定表生成
		 .designatedTableName(new ArrayList<>())
		 //根据表前缀生成
		 .designatedTablePrefix(new ArrayList<>())
		 //根据表后缀生成	
		 .designatedTableSuffix(new ArrayList<>())
         //忽略表名
         .ignoreTableName(ignoreTableName)
         //忽略表前缀
         .ignoreTablePrefix(ignorePrefix)
         //忽略表后缀
         .ignoreTableSuffix(ignoreSuffix).build();
   //配置
   Configuration config = Configuration.builder()
         //版本
         .version("1.0.0")
         //描述
         .description("数据库设计文档生成")
         //数据源
         .dataSource(dataSource)
         //生成配置
         .engineConfig(engineConfig)
         //生成配置
         .produceConfig(processConfig)
         .build();
   //执行生成
   new DocumentationExecute(config).execute();
}

```

## 方式二 直接pom操作
```
<build>
		<plugins>
			<plugin>
				<groupId>cn.smallbun.screw</groupId>
				<artifactId>screw-maven-plugin</artifactId>
				<version>1.0.5</version>
				<dependencies>
					<!-- HikariCP -->
					<dependency>
						<groupId>com.zaxxer</groupId>
						<artifactId>HikariCP</artifactId>
						<version>3.4.5</version>
					</dependency>
					<!--mysql driver-->
					<dependency>
						<groupId>mysql</groupId>
						<artifactId>mysql-connector-java</artifactId>
						<version>8.0.20</version>
					</dependency>
				</dependencies>
				<configuration>
					<!--username-->
					<username>root</username>
					<!--password-->
					<password>11</password>
					<!--driver-->
					<driverClassName>com.mysql.cj.jdbc.Driver</driverClassName>
					<!--jdbc url-->
					<jdbcUrl>jdbc:mysql://127.0.0.1:3306/xxxx</jdbcUrl>
					<!--生成文件类型-->
					<fileType>HTML</fileType>
					<!--打开文件输出目录-->
					<openOutputDir>true</openOutputDir>
					<!--生成模板-->
					<produceType>freemarker</produceType>
					<!--文档名称 为空时:将采用[数据库名称-描述-版本号]作为文档名称-->
					<fileName>测试文档名称</fileName>
					<!--描述-->
					<description>数据库文档生成</description>
					<!--版本-->
					<version>${project.version}</version>
					<!--标题-->
					<title>数据库文档</title>
				</configuration>
				<executions>
					<execution>
						<phase>compile</phase>
						<goals>
							<goal>run</goal>
						</goals>
					</execution>
				</executions>
			</plugin>
		</plugins>
	</build>
```
