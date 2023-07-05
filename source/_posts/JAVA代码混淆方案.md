---
title: JAVA代码混淆方案
date: 2023-07-05 16:41:15
tags: [java]
---
# ClassFinal

<https://toscode.gitee.com/roseboy/classfinal/>

在编译的时候加入生成一个加密的后的jar包

需要指定命令运行

<!--more-->

### 普通jar包加密 有密码和无密码启动
```
    无密码:
    java -javaagent:guiDemo-0.0.1-SNAPSHOT-encrypted.jar  -jar .\guiDemo-0.0.1-SNAPSHOT-encrypted.jar

    有密码:
    java -javaagent:guiDemo-0.0.1-SNAPSHOT-encrypted.jar='-pwd 0000000' -jar guiDemo-0.0.1-SNAPSHOT-encrypted.jar
```
### war包加密 有密码和无密码启动
```
    tomcat/bin/catalina 增加以下配置:

    //linux下 catalina.sh
    CATALINA_OPTS="$CATALINA_OPTS -javaagent:classfinal-fatjar.jar='-pwd 0000000'";
    export CATALINA_OPTS;

    //win下catalina.bat
    set JAVA_OPTS="-javaagent:classfinal-fatjar.jar='-pwd 000000'"

    //参数说明 
    // -pwd      加密项目的密码  
    // -nopwd    无密码加密时启动加上此参数，跳过输密码过程
    // -pwdname  环境变量中密码的名字
```

# 使用方式

maven引入, 注意这个部分需要放到原本的spring-boot-maven-plugin之后

直接使用clean 和package 可生成
```
                <plugin>
                    <groupId>net.roseboy</groupId>
                    <artifactId>classfinal-maven-plugin</artifactId>
                    <version>1.2.1</version>
                    <configuration>
                        <password>#</password><!-- #表示启动时不需要密码，事实上对于代码混淆来说，这个密码没什么用，它只是一个启动密码 -->
                        <packages>com.linjingc.guidemo</packages><!-- 加密的包名，多个包用逗号分开-->
                        <excludes>org.spring</excludes>
                    </configuration>
                    <executions>
                        <execution>
                            <phase>package</phase>
                            <goals>
                                <goal>classFinal</goal>
                            </goals>
                        </execution>
                    </executions>
                </plugin>
```
