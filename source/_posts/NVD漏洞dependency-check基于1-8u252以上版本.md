---
title: NVD漏洞dependency-check基于1.8u252以上版本
date: 2024-05-04 15:47:11
tags: [SpringBoot, nvd,dependency-check, maven]
---
# 基于maven进行漏洞扫描并且生生成报告
```
   <plugin>
       <groupId>org.owasp</groupId>
       <artifactId>dependency-check-maven</artifactId>
       <version>9.0.10</version>
       <configuration>
           <autoUpdate>true</autoUpdate>
       </configuration>
       <executions>
           <execution>
               <goals>
                   <goal>check</goal>
               </goals>
           </execution>
       </executions>
   </plugin>

```
<!--more-->
依赖我放在百度云上了

仓库的内容 dependency-check-maven.zip
