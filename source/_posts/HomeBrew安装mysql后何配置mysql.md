---
layout: homebrew
title: HomeBrew安装mysql后，如何配置mysql
date: 2019-01-14 15:06:03
tags: [HomeBrew,mysql]
---



# 安装mysql

brew install mysql 

可以指定版本

brew install mysql@5.7

<!--more-->

# [启动]Mysql服务

mysql.server start

关闭

mysql.server stop

# mysql配置脚本完成初始化

使用mysql的配置脚本：/usr/local/opt/mysql/bin/mysql_secure_installation //mysql 提供的配置向导 

启动这个脚本后，即可根据如下命令提示进行初始化设置

```
sunyichaodeMacBook-Pro:~ sunyichao$ /usr/local/opt/mysql/bin/mysql_secure_installation   //mysql 提供的配置向导
NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MySQL
      SERVERS IN PRODUCTION USE!  PLEASE READ EACH STEP CAREFULLY!

In order to log into MySQL to secure it, we'll need the current
password for the root user.  If you've just installed MySQL, and
you haven't set the root password yet, the password will be blank,
so you should just press enter here.

Enter current password for root (enter for none): //输入现行root密码，因为初次使用，所以直接回车
OK, successfully used password, moving on...

Setting the root password ensures that nobody can log into the MySQL
root user without the proper authorisation.

Set root password? [Y/n] Y //是否设置root密码
New password:
Re-enter new password:
Password updated successfully!
Reloading privilege tables..
 ... Success!

By default, a MySQL installation has an anonymous user, allowing anyone to log into MySQL without having to have a user account created for This is intended only for testing, and to make the installation go a bit smoother. You should remove them before moving into a production environment.    
Remove anonymous users? [Y/n] Y //是否删除匿名用户
 ... Success!

Normally, root should only be allowed to connect from 'localhost'.This
ensures that someone cannot guess at the root password from the network.

Disallow root login remotely? [Y/n] Y //是否禁止远程登录
 ... Success!

By default, MySQL comes with a database named 'test' that anyone can
access.This is also intended only for testing, and should be removed
before moving into a production environment.

Remove test database and access to it? [Y/n] Y //删除测试数据库，并登录
 Dropping test database...
 ... Success!
 Removing privileges on test database...
 ... Success!

Reloading the privilege tables will ensure that all changes made so far
will take effect immediately.

Reload privilege tables now? [Y/n] Y//重新载入权限表
 ... Success!

All done!  If you've completed all of the above steps, your MySQL
installation should now be secure.

Thanks for using MySQL!

Cleaning up...
sunyichaodeMacBook-Pro:~ sunyichao$
```



# 设置开机启动

```
mkdir -p ~/Library/LaunchAgents
cp /usr/local/Cellar/mysql/<Mysql版本号>/homebrew.mxcl.mysql.plist ~/Library/LaunchAgents/
launchctl load -w ~/Library/LaunchAgents/homebrew.mxcl.mysql.plist
```



