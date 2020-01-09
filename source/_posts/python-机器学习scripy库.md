---
title: python-机器学习scripy库
date: 2020-01-09 20:08:43
tags: [python,机器学习]
---

# scripy 数值计算库

在numpy在新增了很多方法

## 官网 www.scipy.org



## demo

```
https://github.com/AsummerCat/scripy_demo
```



## 积分 integrate

<!--more-->

```python
# -*- coding: utf-8 -*-
import numpy as np

'''
积分
'''
from scipy.integrate import quad, dblquad, nquad


def test():
    # 积分       范围 ,0,无权大
    # 获取结果 值 和偏差值 取值 为 值+ -误差
    print(quad(lambda x: np.exp(-x), 0, np.inf))
    print("=======================================")

    # 二元积分        t的范围可定义常数 x用表达式
    # 获取结果 值 和偏差值 取值 为 值+ -误差
    print(dblquad(lambda t, x: np.exp(-x * t) / t ** 3, 0, np.inf, lambda x: 1, lambda x: np.inf))
    print("=======================================")

    # n维积分
    def f(x, y): return x * y

    ## y的边界 和x的边界
    def bound_y():
        return [0, 0.5]

    def bound_x(y):
        return [0, 1 - 2 * y]

    print(nquad(f, [bound_x, bound_y]))
    print("=======================================")


if __name__ == '__main__':
    test()

```

## 优化器 optimize

```python
# -*- coding: utf-8 -*-

'''
优化器
'''

from scipy.integrate import quad, dblquad, nquad
import numpy as np
# 计算最小值
from scipy.optimize import minimize


def test():
    # 无约束最小化多元标量函数 Nelder-Mead（单纯形法）
    def rosen(x):
        return sum(100.0 * (x[1:] - x[:-1] ** 2.0) ** 2.0 + (1 - x[:-1]) ** 2.0)

    def callback(xk):
        print(xk)

    x0 = np.array([1.3, 0.7, 0.8, 1.9, 1.2])
    # 容忍精度，是否打印
    options = {"xto1": 1e-8, "disp": True}
    # 计算最小点 根据rosen 获取最小点 算法:1度5点为基础 5个点合拢成最小值
    res = minimize(rosen, x0, method='nelder-mead', options=options, callback=callback)
    print("ROSE MINI", res)

    pass


if __name__ == '__main__':
    test()
    pass

```

### 求根

```python
def test1():
    from scipy.optimize import root
    def fun(x):
        return x + 2 * np.cos(x)

    sol = root(fun, 0.1)
    print("ROOT", sol.x, sol.fun)
```

## 插值 interpolate

```python
# -*- coding: utf-8 -*-
'''
插值
'''
# 绘图模块
from pylab import *
import numpy as np
from scipy.interpolate import interp1d


def test():
    # 插值算法
    x = np.linspace(0, 1, 10)  # 产生0-1之间10个数
    y = np.sin(2 * np.pi * x)  # 指定函数
    # 不加入参数为线性插值
    # # li=interp1d(x,y)
    li = interp1d(x, y, kind="cubic")  # 定义一个三阶函数曲线插值

    x_new = np.linspace(0, 1, 50)  # 定义0-1 50个数
    y_new = li(x_new)  # 获取结果

    figure()  # 画出来
    plot(x, y, "r")  # 用红色表示原数据
    plot(x_new, y_new, "k")  # 用黑色表示新数据
    show()


if __name__ == '__main__':
    test()

```

## 线性函数 Linear

矩阵分析

```python
# -*- coding: utf-8 -*-
'''
线性函数处理
'''

import numpy as np
# 线性处理模块
from scipy import linalg as lg


def test():
    # 矩阵
    arr = np.array([[1, 2], [3, 4]])
    ## 计算行列式
    print("Det:", lg.det(arr))
    ## 计算矩阵求逆
    print("Inv:", lg.inv(arr))

    # 解线性方程组
    b = np.array([6, 14])
    print("Sol:", lg.solve(arr, b))
    # 特征值
    print("Eig:", lg.eig(arr))
    # 矩阵分解
    ## LU分解
    print("LU:", lg.lu(arr))
    ## QR分解
    print("QR:", lg.qr(arr))
    ## 奇异值分解
    print("SVD:", lg.svd(arr))
    ## 舒尔分解
    print("Schur:", lg.schur(arr))

    pass


if __name__ == '__main__':
    test()

```

