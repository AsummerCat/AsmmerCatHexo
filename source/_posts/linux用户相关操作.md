---
title: linux用户相关操作
date: 2019-12-16 22:56:13
tags: [linux]
---

# linux用户

## 添加用户账号 

```linux
useradd [参数] 新建用户账户 -m
adduser [参数] 新建用户账户 -m

-d 指定用户登录系统时的主目录  -->默认/home下同名文件夹为主目录
-m 自动建立目录
-g 指定组名称  不指定组名自动创建一个与用户名相同的组
```

## 设置用户名密码

```
 passwd
```

### root用户下操作 可修改任意用户密码

```
passwd abc
输入新密码
确认新密码
```

### 普通用户操作

```
passwd
修改当前用户密码 
```



<!--more-->

## 删除用户

```
 userdel 用户名             ->删除用户 不删除用户主目录
 userdel -r 用户名          ->删除用户 并且删除用户主目录
```



## 切换用户

```
su 用户名           不切换当前路径
su - 用户名         切换当前路径
su -s              切换到root用户

```



## 当前用户获取权限

```
sudo 动作 
```



## 查看当前用户

```
whoami
```

# 用户组

## 添加用户组

```
groupadd 新建组账号
```



## 删除用户组

```
groupdel 删除组账号
```

## 查看用户组

```
cat /etc/group
```



## 修改用户所在组

```
usermod -g 用户组 用户名
```



# chomd权限

  写法: chomd u/g/o/a

```
u:表示文件所有者

g= group 用户组

o: 其他人

a: all 三者皆是
```

权限:

```
rwx:

r: 读

w:写

x: 可执行
```

## 测试语句

### 更改文件权限

```
chomd u=r,g=r,o=x xxx文件
chomd u=rwx xxx文件
```

### 新增删除权限

```
chomd u+w xxx文件
chomd u-w xxx文件
chomd 777 xxx文件   开启所有权限
chome 000 xxx文件   关闭所有权限  二进制
```



##  修改文件拥有者:chown

chome 修改的后的用户 文件名称

```
chown edu 1.text
```





## 修改文件归属组 :chgrp

chgrp 修改后的组 文件名称

```
chgrp root 1.text
```









