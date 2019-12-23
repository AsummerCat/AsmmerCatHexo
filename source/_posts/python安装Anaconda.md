---
title: python安装Anaconda
date: 2019-12-13 11:36:03
tags: [python]
---

# Anaconda

一款开源的软件包 

如果直接安装python包 第三方插件需要手动下载

Anaconda中自带了很多第三方的包

## 安装

地址:https://www.anaconda.com/

可以直接用迅雷下载速度很快

## 安装过程

安装过程中选择 两个√

第一个 添加路径到path

第二个 安装默认的python环境

这样继续下一步 

完成安装后

## 输入python 查看版本号

```linux
python
```

## 输入conda命令

```
conda
```

## Python环境需要激活

<!--more-->

当能够使用conda命令后，就需要解决Python环境未激活的问题。首先在cmd中输入命令

```python
conda info --envs
```

# Conda的包管理

Conda的包管理就比较好理解了，这部分功能与`pip`类似。

例如，如果需要安装scipy：

```
# 安装scipy
conda install scipy
# conda会从从远程搜索scipy的相关信息和依赖项目，对于python 3.4，conda会同时安装numpy和mkl（运算加速的库）
 
# 查看已经安装的packages
conda list
# 最新版的conda是从site-packages文件夹中搜索已经安装的包，不依赖于pip，因此可以显示出通过各种方式安装的包
```

conda的一些常用操作如下：

```
# 查看当前环境下已安装的包
conda list
 
# 查看某个指定环境的已安装包
conda list -n python34
 
# 查找package信息
conda search numpy
 
# 安装package
conda install -n python34 numpy
# 如果不用-n指定环境名称，则被安装在当前活跃环境
# 也可以通过-c指定通过某个channel安装
 
# 更新package
conda update -n python34 numpy
 
# 删除package
conda remove -n python34 numpy

```

前面已经提到，conda将conda、python等都视为package，因此，完全可以使用conda来管理conda和python的版本，例如

```
# 更新conda，保持conda最新
conda update conda
 
# 更新anaconda
conda update anaconda
 
# 更新python
conda update python
# 假设当前环境是python 3.4, conda会将python升级为3.4.x系列的当前最新版本
```

## 设置国内镜像

如果需要安装很多packages，你会发现conda下载的速度经常很慢，因为Anaconda.org的服务器在国外。所幸的是，清华TUNA镜像源有Anaconda仓库的镜像，我们将其加入conda的配置即可：

```
# 添加Anaconda的TUNA镜像
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
# TUNA的help中镜像地址加有引号，需要去掉
 
# 设置搜索时显示通道地址
conda config --set show_channel_urls yes

```

## 导入外部的包 如何安装

例如qrCode

### 方式一

直接访问 `https://anaconda.org/` 进行搜索需要的包

```
conda install -c conda-forge qrcode
```

## 方式二

```
anaconda search -t conda  需要的包名
```

```
anaconda show 上面搜索到的名称
```

搜索到 最后一行 会获取一个地址 下载就可以了

```
conda install --channel https://conda.anaconda.org/conda-forge pydap
```



# 问题解决

## python无法使用

```
 控制台输入 python
```

如果无法输出

手动配置用户环境变量

添加 Anaconda安装路径 ->path中

(Anaconda根目录下有个 python.exe)

## conda 命令无法使用

> ‘conda’ 不是内部或外部命令，也不是可运行的程序或批处理文件

这时，只要把anaconda目录下的scripts文件夹添加到环境变量中就行了。

Python环境需要激活
当能够使用conda命令后，就需要解决Python环境未激活的问题。首先在cmd中输入命令

`conda info --envs`
1
会出现这种效果，此图是从其它地方找的，并不是我的截图，但是显示的结果是相同的。

下面显示的就是要激活的路径。
注意，这里有个Warning，这个是conda的版本问题，执行下面命令后即可消除：

`conda update conda`

然后，执行命令

`conda activate myenv`

myenv为执行上一条命令后得到的结果，执行完毕后，之前的警告就能够消除了。
注意，这里有一个问题，就是你必须在每次重新打开cmd后先输入activate命令，否则还是会显示python未激活，这个问题我现在也不知道怎么解决，希望有知道的小伙伴能讲下，要不直接卸了重装python也行。
