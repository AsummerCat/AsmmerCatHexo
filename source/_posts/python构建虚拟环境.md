---
title: python构建虚拟环境
date: 2019-12-17 09:30:12
tags: [python]
---

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