---
title: docker集成maven构建springboot项目
date: 2019-08-03 17:26:49
tags: [docker,maven,SpringBoot]
---

# docker集成maven构建springboot项目 

暂时基本可用 后期再学习

[demo地址](https://github.com/AsummerCat/springboot-docker)

## 需要注意的是构建的项目名称必须是小写 不然会编译不过

## ADD failed: stat /var/lib/docker/tmp/docker-builderXXXXXX: no such file or directory

```
ADD user-server-0.0.1-SNAPSHOT.jar app.jar  要和pom的<artifactId>user-server</artifactId>

保持名字一样，不然maven打出来的包，docker找不到。
```

<!--more-->

## Failed to execute goal com.spotify:docker-maven-plugin:0.4.3:build (default) on project users-microservice: Exception caught: java.util.concurrent.ExecutionException: com.spotify.docker.client.shaded.javax.ws.rs.ProcessingException: org.apache.http.conn.HttpHostConnectException: Connect to localhost:2375 [localhost/127.0.0.1, localhost/0:0:0:0:0:0:0:1] failed: Connection refused -> [Help 1]

```
docker-maven-plugin的version从0.4.3update到1.0.0就可以了！
```



# 开始构建

## 首先加入构建docker的打包程序 记住要放在spring-boot-maven-plugin之后

```
<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
			<plugin>
				<groupId>com.spotify</groupId>
				<artifactId>docker-maven-plugin</artifactId>
				<version>1.0.0</version>
				<configuration>
					<imageName>${project.artifactId}</imageName>
					<dockerDirectory>src/main/docker</dockerDirectory>
					<resources>
						<resource>
							<targetPath>/</targetPath>
							<directory>${project.build.directory}</directory>
							<include>${project.build.finalName}.jar</include>
						</resource>
					</resources>
				</configuration>
			</plugin>
			</plugins>
	</build>
```

## 第二步 添加Dockerfile

路径位置 /sec/mian/docker/Dockerfile

```
FROM frolvlad/alpine-oraclejdk8:slim
VOLUME /tmp
ADD testdemo-0.0.1-SNAPSHOT.jar app.jar
#RUN bash -c 'touch /app.jar'
ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]
EXPOSE 8761
```

ADD这个是要写你项目的名称

## 第三步 maven 打包  

打包后 在运行 maven集成docker的docker:build