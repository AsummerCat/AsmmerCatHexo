---
title: SpringBoot使用batch批处理
date: 2023-01-11 11:11:41
tags: [SpringBoot,SpringBatch]
---

Spring Batch为XML，Flat文件，CSV，MYSQL，Hibernate，JDBC，Mongo，Neo4j等大量写入器和读取器提供支持。

## demo地址
```
https://spring.io/guides/gs/batch-processing/

https://github.com/AsummerCat/SpringBatchDemo.git
```
## 导入相关依赖
```
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-batch</artifactId>
        </dependency>
```
<!--more-->

## 注意需要初始化脚本 来存储batch的元数据
```
\spring-batch\spring-batch-core\src\main\resources\org\springframework\batch\core\schema-mysql.sql  

路径在这里
```

## job任务完成通知
```
package com.linjingc.springbatchdemo.batch;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.batch.core.BatchStatus;
import org.springframework.batch.core.JobExecution;
import org.springframework.batch.core.listener.JobExecutionListenerSupport;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.stereotype.Component;

/**
 * job任务完成通知
 */
@Component
public class JobCompletionNotificationListener extends JobExecutionListenerSupport {

    private static final Logger log = LoggerFactory.getLogger(JobCompletionNotificationListener.class);
    @Autowired
    private JdbcTemplate jdbcTemplate;

    @Override
    public void afterJob(JobExecution jobExecution) {
        if (jobExecution.getStatus() == BatchStatus.COMPLETED) {
            log.info("!!! JOB FINISHED! Time to verify the results");

            jdbcTemplate.query("SELECT first_name, last_name FROM people",
                    (rs, row) -> new Person(
                            rs.getString(1),
                            rs.getString(2))
            ).forEach(person -> log.info("Found <" + person + "> in the database."));
        }
    }
}
```

## 作业任务
注意开启`@EnableBatchProcessing
`


## 实体
```
package com.linjingc.springbatchdemo.batch;

import lombok.AllArgsConstructor;
import lombok.Data;

/**
 * 实体
 */
@Data
@AllArgsConstructor
public class Person {

    private String lastName;
    private String firstName;
}
```
```
p

## ackage com.linjingc.springbatchdemo.batch;

import org.springframework.batch.core.Job;
import org.springframework.batch.core.Step;
import org.springframework.batch.core.configuration.annotation.EnableBatchProcessing;
import org.springframework.batch.core.configuration.annotation.JobBuilderFactory;
import org.springframework.batch.core.configuration.annotation.StepBuilderFactory;
import org.springframework.batch.core.launch.support.RunIdIncrementer;
import org.springframework.batch.item.ItemReader;
import org.springframework.batch.item.ItemWriter;
import org.springframework.batch.item.database.BeanPropertyItemSqlParameterSourceProvider;
import org.springframework.batch.item.database.JdbcBatchItemWriter;
import org.springframework.batch.item.database.builder.JdbcBatchItemWriterBuilder;
import org.springframework.batch.item.file.FlatFileItemReader;
import org.springframework.batch.item.file.builder.FlatFileItemReaderBuilder;
import org.springframework.batch.item.file.mapping.BeanWrapperFieldSetMapper;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Import;
import org.springframework.core.io.ClassPathResource;

import javax.sql.DataSource;


@Configuration
@EnableBatchProcessing
//@Import(DataSourceConfiguration.class)
public class JobTask {
    @Autowired
    private JobBuilderFactory jobs;
    @Autowired
    private StepBuilderFactory steps;

    @Bean
    public Job importUserJob(JobCompletionNotificationListener jobCompletionNotificationListener, @Qualifier("step1") Step step1) {
        return jobs.get("importUserJob")
                .incrementer(new RunIdIncrementer())
                .listener(jobCompletionNotificationListener)
                .flow(step1)
                .end()
                .build();
    }


    @Bean
    protected Step step1(ItemReader<Person> reader,
                         PersonItemProcessor processor,
                         ItemWriter<PersonNext> writer) {
        return steps.get("step1")
                //这里有一个chunk的设置，值10，意思是10条记录后再提交输出
                .<Person, PersonNext>chunk(10)
                .reader(reader)
                .processor(processor)
                .writer(writer)
                .build();
    }


    /**
     * 输入 读取文本内容.CSV
     */
    @Bean
    public FlatFileItemReader<Person> reader() {
        return new FlatFileItemReaderBuilder<Person>()
                .name("personItemReader")
                .resource(new ClassPathResource("sample-data.csv"))
                //跳过CSV第一行,表头
                .linesToSkip(1)
                .delimited()
                //字段名
                .names(new String[]{"id,", "firstName", "lastName"})
                .fieldSetMapper(new BeanWrapperFieldSetMapper<Person>() {{
                    //转换后的目标类
                    setTargetType(Person.class);
                }})
                .build();
    }


    /**
     * 处理器
     *
     * @return
     */
    @Bean
    public PersonItemProcessor processor() {
        return new PersonItemProcessor();
    }


    /**
     * 输出
     *
     * @param dataSource
     * @return
     */
    @Bean
    public JdbcBatchItemWriter<PersonNext> writer(DataSource dataSource) {
        return new JdbcBatchItemWriterBuilder<PersonNext>()
                .itemSqlParameterSourceProvider(new BeanPropertyItemSqlParameterSourceProvider<>())
                .sql("INSERT INTO people (first_name, last_name) VALUES (:firstName, :lastName)")
                .dataSource(dataSource)
                .build();
    }
}

```

## 实体
```
package com.linjingc.springbatchdemo.batch;

import lombok.AllArgsConstructor;
import lombok.Data;

/**
 * 实体
 */
@Data
@AllArgsConstructor
public class Person {

    private String lastName;
    private String firstName;
}

```
```
package com.linjingc.springbatchdemo.batch;

import lombok.AllArgsConstructor;
import lombok.Data;

/**
 * 实体
 */
@Data
@AllArgsConstructor
public class PersonNext {

    private String lastName;
    private String firstName;
}
```
## 转换器
```
package com.linjingc.springbatchdemo.batch;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.batch.item.ItemProcessor;

/**
 * ItemProcessor 转换器
 * Person ->转换为 PersonNext
 */
public class PersonItemProcessor implements ItemProcessor<Person, PersonNext> {

    private static final Logger log = LoggerFactory.getLogger(PersonItemProcessor.class);

    @Override
    public PersonNext process(final Person person) {
        final String firstName = person.getFirstName().toUpperCase();
        final String lastName = person.getLastName().toUpperCase();

        final PersonNext transformedPerson = new PersonNext(firstName, lastName);

        log.info("Converting (" + person + ") into (" + transformedPerson + ")");

        return transformedPerson;
    }

}
```