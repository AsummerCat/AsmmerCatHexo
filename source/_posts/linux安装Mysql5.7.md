---
title: linux安装Mysql5.7
date: 2018-09-26 22:56:22
tags: [mysql,数据库,linux]
---
>tips:从CentOS 7.0发布以来，yum源中开始使,用Mariadb来代替MySQL的安装。  
>即使你输入的是yum install -y mysql , 显示的也是Mariadb的安装内容。使用源代码进行编译安装又太麻烦。  
>因此，如果想使用yum安装MySQL的话，就需要去下载官方指定的yum源  
>网址为： 
>https://dev.mysql.com/downloads/repo/yum/   
>找到Red Hat Enterprise Linux 7 / Oracle Linux 7 (Architecture Independent), RPM Package,
>单击后面的Download，在新的页面中单击最下面的No thanks, just start my download.就可以下载到yum源了.

<!--more-->
---
---
---



### 首先进入本机的源文件目录
```
cd /usr/local/src
```
### 使用wget下载官方yum源的rpm包

`wget https://dev.mysql.com/get/mysql57-community-release-el7-11.noarch.rpm`

### 安装rpm包
```
rpm -ivh mysql57-community-release-el7-11.noarch.rpm
```

### 再次使用yum来安装mysql-server
```
yum install -y mysql-server
可以看到这次不再提示安装Mariadb了
```
### 注意 需要关闭mysql大小写敏感 默认是开启的
```
lower_case_table_names=1  
```

### 安装完成后，启动mysqld服务
```
systemctl start mysqld
查看是否成功启动：

ps aux|grep mysqld
```

### 设置mysqld服务开机自启动
```
systemctl enable mysqld
```

### 使用初始密码登录
`登陆后,这个可以查询初始密码存放位置`

>由于MySQL从5.7开始不允许首次安装后，使用空密码进行登录，系统会随机生成一个密码以供管理员首次登录使用，这个密码记录在/var/log/mysqld.log文件中，使用下面的命令可以查看此密码：

`
cat /var/log/mysqld.log|grep 'A temporary password'
`  
>
2017-11-12T13:35:37.013617Z 1 [Note] A temporary password is generated for root@localhost: bkv,dy,)o7Ss
最后一行冒号后面的部分bkv,dy,)o7Ss就是初始密码。 
使用此密码登录MySQL:  

`
mysql -u root -p
`

---
### 更改默认密码

#### 修改root密码：

`
alter user 'root'@'localhost' identified by 'your_password'; 
 `

>将your_password替换成你自己的密码就可以了，当然，这个密码是强密码，要求密码包含大小写字母、数字及标点符号，长度应该在6位以上。 

重新使用新的密码登录，如果可以正常登录说明你的MySQL已经成功安装在CentOS 7.4上了

#### 用该密码登录到服务端后，必须马上修改密码，不然会报如下错误：
>ERROR 1820 (HY000): You must reset your password using ALTER USER statement before executing this statement.

#### 如果只是修改为一个简单的密码，会报以下错误：
>ERROR 1819 (HY000): Your password does not satisfy the current policy requirements  

`要求密码包含大小写字母、数字及标点符号，长度应该在6位以上。`  

---

### 授权主机登陆
#### 授权任意主机可登录
`GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '你的密码' WITH GRANT OPTION; 
` 

#### 授权指定主机可登录 
`GRANT ALL PRIVILEGES ON *.* TO 'root'@'xxx.xxx.xxx.xx' IDENTIFIED BY '你的密码' WITH GRANT OPTION;`

#### 立即执行新权限

`FLUSH PRIVILEGES;`  

#### 退出MySQL命令行
`quit`

---

### 重启服务
```
使用如下命令操作mysql： 
systemctl restart mysqld.service    重启
systemctl start mysqld.service 		启动
systemctl stop mysqld.service	    关闭
```

### 开放3306端口
```
centos7 在新安装的Linux系统中，防火墙默认是被禁掉的，一般也没有配置过任何防火墙的策略，  
所以不存在/etc/sysconfig/iptables文件。

```

