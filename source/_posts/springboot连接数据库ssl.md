---
title: springboot连接数据库ssl
date: 2022-11-17 08:27:45
tags: [SpringBoot,mysql,数据库]
---
# springboot连接数据库ssl

## 首先数据库导出
```
ca.pem
client-cert.pem
client-key.pem
```

## 1.首先查询linux上的openssl版本
```
openssl version -a
```

## 2.升级openssl版本
从openssl官网下载最新的稳定版本，https://www.openssl.org/source/ 当前的稳定版是  	openssl-3.0.7.tar.gz（联邦信息处理标准（Federal Information Processing Standards，FIPS）是一套描述文件处理、加密算法和其他信息技术标准（在非军用政府机构和与这些机构合作的政府承包商和供应商中应用的标准）的标准。），下载后上传到服务器的/usr/local/src目录下。
```
cd /usr/local/src 
## 检查是否有gcc编辑器
gcc -v
yum update gcc

## 解压openssl包
tar -xzf openssl-3.0.7.tar.gz

## 检查是否已安装zlib库
whereis zlib
## 安装zlib库
yum -y install zlib

## 安装
yum install -y perl-CPAN
进入 /root/.cpan/CPAN/修改MYConfig.pm
  'urllist' => [q[https://mirror.tuna.tsinghua.edu.cn/CPAN/]],
再执行  
perl -MCPAN -e shell
install IPC/Cmd.pm




## 安装openssl到 /usr/local/openssl 目录
./config shared zlib  --prefix=/usr/local/openssl && make && make install

```
<!--more-->



使用源码按过于繁琐，如果对软件版本没有特殊要求的话可以使用yum命令安装和更新，既方便又快捷

yum install openssl
yum update openssl


# 分为ssl单向和双向连接

## 如果数据库仅仅开启`require_secure_transport=yes`
并没有对指定用户开启ssl登录
```
那么直接在jdbc连接后面加上

useSSL=true&verifyServerCertificate=false
即可
```
这样表示开启SSL认证并且忽略客户端证书验证
注意：`verifyServerCertificate=false`不需要验证客户端密钥

## 单向:ca.pem mysql服务器上获取
```
keytool -import -trustcacerts -v -alias Mysql -file "C:\ProgramData\MySQL\MySQL Server 8.0\Data\ca.pem" -keystore "mysql.jks"
设置密钥并记住密钥
```
验证证书是否导入
```
keytool -list -keystore mysql.jks
```
使用
```
ssl:
  cert:
    path: D:/Java/jdk1.8.0_131/bin
  config:
      url: true&trustCertificateKeyStorePassword=123456&trustCertificateKeyStoreUrl=file:${ssl.cert.path}/mysql.jks
spring:
  datasource:
      driver-class-name: com.mysql.jdbc.Driver
      url: jdbc:mysql://127.0.0.1:3316/db1?useUnicode=true&characterEncoding=utf8&allowMultiQueries=true&useSSL=${ssl.config.url}
      username: ch
      password: ch123456
```

## 双向:
前置条件需要这三个pem 数据库上拿
```
ca.pem
client-cert.pem
client-key.pem
```
生成对应证书
```
设置服务器身份验证：导入服务器证书
keytool -importcert -alias MySQLCACert -file ca.pem -keystore truststore.jks -storepass 123456


设置客户端身份验证：
将客户端密钥和证书文件转换为 PKCS #12 存档：(在数据库服务器生成)
openssl pkcs12 -export -in client-cert.pem -inkey client-key.pem -name "mysqlclient" -passout pass:123456 -out client-keystore.p12

将客户端密钥和证书导入 Java 密钥库：
keytool -importkeystore -srckeystore client-keystore.p12 -srcstoretype pkcs12 -srcstorepass 123456 -destkeystore keystore.jks -deststoretype JKS -deststorepass 123456

```
使用:
```
server:
  port: 8300

ssl:
  cert:
    path: D:/Java/jdk1.8.0_131/bin
  config:
    url: true&trustCertificateKeyStorePassword=123456&trustCertificateKeyStoreUrl=file:${ssl.cert.path}/truststore.jks&&clientCertificateKeyStoreUrl=file:${ssl.cert.path}/keystore.jks&clientCertificateKeyStorePassword=123456
spring:
  datasource:
    driver-class-name: com.mysql.jdbc.Driver
    url: jdbc:mysql://127.0.0.1:3306/db1?useUnicode=true&characterEncoding=utf8&allowMultiQueries=true&useSSL=${ssl.config.url}
    username: ch
    password: ch123456
```

