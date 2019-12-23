---
title: python构建虚拟环境
date: 2019-12-17 09:30:12
tags: [python]
---

# Anaconda 构建虚拟环境

## 查看当前所有虚拟环境

```
conda env list
```

## 创建虚拟环境

```
conda create -n my_env
```

* 这里有个小技巧：如果是要指定目录创建虚拟环境到
  `conda create --prefix=D:\WebApp\Projects\djangoproject`

## 进入虚拟环境

```
activate my_env
或者
activate D:\WebApp\Projects\djangoproject
```

此时我们可以看到，命令行的左侧多出了一个（my_env），代表我们当前是在该环境下进行命令行的操作。如果我们此时再输入：`conda env list`，可以看到星号（*）已经移到了刚刚创建的虚拟环境目录的左侧。需要注意的是，如果关闭了Anaconda Promt，再新再打开的话，那么还需要重新进行一次激活操作。

## 退出虚拟环境

```
deactivate
```

## 移除虚拟环境

```
conda remove -n qrCode --all
或
conda remove -n D:\WebApp\Projects\djangoproject --all
```

## 卸载虚拟环境下所有包

```
condat remove -n my_env -all
```



# python 构建虚拟环境

这个工具叫virtualenv，是使用python开发的一个创建虚拟环境的工具，源码官网地址：https://github.com/pypa/virtualenv

## 安装 

```python
pip install virtualenv
```

创建一个虚拟环境：virtualenv env1

创建并进入环境：mkvirtualenv env1
退出环境：deactivate
进入已存在的环境或者切换环境：workon env1或者env2
删除环境： rmvirtualenv env1