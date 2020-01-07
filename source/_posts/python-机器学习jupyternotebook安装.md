---
title: python-机器学习jupyternotebook安装
date: 2020-01-07 13:50:46
tags: [python,机器学习]
---

# jupyter notebook

Jupyter Notebook 是一款开放源代码的 Web 应用程序，可让我们创建并共享代码和文档。

jupyter notebook安装_飞桨PaddlePaddle-开源深度学习平台

<!--more-->

## 1. 安装

### ① 安装前提

安装Jupyter Notebook的前提是需要安装了Python（3.3版本及以上，或2.7版本）。

### ② 使用Anaconda安装

如果你是小白，那么建议你通过安装Anaconda来解决Jupyter Notebook的安装问题，因为Anaconda已经自动为你安装了Jupter Notebook及其他工具，还有python中超过180个科学包及其依赖项。

你可以通过进入Anaconda的[官方下载页面](https://link.jianshu.com?t=https%3A%2F%2Fwww.anaconda.com%2Fdownload%2F%23macos)自行选择下载；如果你对阅读**英文文档**感到头痛，或者对**安装步骤**一无所知，甚至也想快速了解一下**什么是Anaconda**，那么可以前往我的另一篇文章[Anaconda介绍、安装及使用教程](https://link.jianshu.com?t=https%3A%2F%2Fzhuanlan.zhihu.com%2Fp%2F32925500)。你想要的，都在里面！

常规来说，安装了Anaconda发行版时已经自动为你安装了Jupyter Notebook的，但如果没有自动安装，那么就在终端（Linux或macOS的“终端”，Windows的“Anaconda Prompt”，以下均简称“终端”）中输入以下命令安装：

```
conda install jupyter notebook
```

### ③ 使用pip命令安装

如果你是有经验的Python玩家，想要尝试用pip命令来安装Jupyter Notebook，那么请看以下步骤吧！接下来的命令都输入在终端当中的噢！

1. 把pip升级到最新版本

   - Python 3.x

   ```
   pip3 install --upgrade pip
   ```

   - Python 2.x

   ```
   pip install --upgrade pip
   ```

- 注意：老版本的pip在安装Jupyter Notebook过程中或面临依赖项无法同步安装的问题。因此**强烈建议**先把pip升级到最新版本。

1. 安装Jupyter Notebook

   - Python 3.x

   ```
   pip3 install jupyter
   ```

   - Python 2.x

   ```
   pip install jupyter
   ```

# 三、运行Jupyter Notebook

## 0. 帮助

如果你有任何jupyter notebook命令的疑问，可以考虑查看官方帮助文档，命令如下：

```
jupyter notebook --help
```

或

```
jupyter notebook -h
```

## 1. 启动

### ① 默认端口启动

在终端中输入以下命令：

```
jupyter notebook
```

执行命令之后，在终端中将会显示一系列notebook的服务器信息，同时浏览器将会自动启动Jupyter Notebook。

启动过程中终端显示内容如下：

```
$ jupyter notebook
[I 08:58:24.417 NotebookApp] Serving notebooks from local directory: /Users/catherine
[I 08:58:24.417 NotebookApp] 0 active kernels
[I 08:58:24.417 NotebookApp] The Jupyter Notebook is running at: http://localhost:8888/
[I 08:58:24.417 NotebookApp] Use Control-C to stop this server and shut down all kernels (twice to skip confirmation).
```

- 注意：之后在Jupyter Notebook的所有操作，都请保持终端**不要关闭**，因为一旦关闭终端，就会断开与本地服务器的链接，你将无法在Jupyter Notebook中进行其他操作啦。

浏览器地址栏中默认地将会显示：`http://localhost:8888`。其中，“localhost”指的是本机，“8888”则是端口号。