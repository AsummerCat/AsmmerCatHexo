---
title: linux安装Mysql8.0
date: 2024-05-26 22:56:22
tags: [mysql,数据库,linux]
---
在CentOS系统上安装MySQL 8.0的步骤稍有不同，因为它使用`yum`作为包管理器。以下是在CentOS上安装MySQL 8.0的详细步骤：

<https://dev.mysql.com/downloads/repo/yum/>

# 1. 添加MySQL Yum Repository

    sudo wget https://dev.mysql.com/get/mysql80-community-release-el7-1.noarch.rpm
    sudo rpm -Uvh mysql80-community-release-el7-1.noarch.rpm

    如果GPG秘钥异常
    sudo rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2023
    sudo yum clean all
    sudo wget https://dev.mysql.com/get/mysql80-community-release-el7-1.noarch.rpm
    sudo rpm -Uvh mysql80-community-release-el7-1.noarch.rpm

<!--more-->
# 2.安装MySQL服务器

```
sudo yum install mysql-community-server

```

# 3.启动MySQL服务

    sudo systemctl start mysqld
    sudo systemctl stop mysqld

    开机启动
    sudo systemctl enable mysqld

<!--more-->

# 4.查找临时生成的root密码

    sudo grep 'temporary password' /var/log/mysqld.log

# 5.安全配置MySQL

    sudo mysql_secure_installation

这个脚本会引导你设置root密码、删除匿名用户、禁止root用户远程登录以及删除测试数据库。你需要输入你找到的临时密码，然后设置一个新的密码，并根据提示完成其他安全设置。

# 6.登录到MySQL

    mysql -u root -p  

# 7.修改数据库端口号

```
sudo find / -name my.cnf

在配置文件中找到[mysqld]部分，然后查找port这一行。如果找不到，你可以添加一行：
port = 3307
//这是关闭大小写敏感
lower_case_table_names=1   
保存

重启
sudo systemctl restart mysqld

验证
sudo netstat -tulnp | grep mysql

```

# 8.外部授权访问

```
1.阿里云 开启安全组

2. 登录 
mysql -u root -p

3.创建用户,授予权限,刷新权限
CREATE USER 'root'@'%' IDENTIFIED BY 'GoodChinaBoy!@1'; 

GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' WITH GRANT OPTION; 

FLUSH PRIVILEGES;

```

