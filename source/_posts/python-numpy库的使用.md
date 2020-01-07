---
title: python-numpy库的使用
date: 2020-01-07 13:56:34
tags: [python,机器学习]
---

# numpy库的使用

https://blog.csdn.net/wxystyle/article/details/80491821

## [demo地址](https://github.com/AsummerCat/numpy_demo)

## [官网api文档](https://docs.scipy.org/doc/)

NumPy是Python科学计算的基础工具包，很多Python数据计算工作库都依赖它

Pandas是一个用于Python数据分析的库，它的主要作用是进行数据分析。Pandas提供用于进行结构化数据分析的二维的表格型数据结构DataFrame，类似于R中的数据框，能提供类似于数据库中的切片、切块、聚合、选择子集等精细化操作，为数据分析提供了便捷

## 安装库

```python
pip install numpy  
```

```python
conda install -c conda-forge numpy
```

<!--more-->

```python
ndarray是N维数组对象（矩阵），其中所有的元素都必须是相同类型。ndarray主要包含以下几个属性：

ndarray.ndim：表示数组对象或矩阵的维度；

ndarray.shape：表示每个维度上的数组的大小；

ndarray.size：表示数组中元素的总数，等同于ndarray.shape中两个元素的乘积；

ndarray.dtype：表示数组中元素的类型；

ndarray.itemsize：表示数组中每个元素的字节大小，比如数据类型为float64的数组，其元素的字节大小为64/8=8。
```

# 测试

```python
import numpy as np
```

### 创建二维数组 三行 一行5个

```python
 # 创建二维数组 三行 一行5个
    a = np.arange(15).reshape(3, 5)
    print(a)
    
  # 创建一个三维矩阵
 a = np.arange(27).reshape(3,3,3)
    print(a)
    ## 获取矩阵大小
    print(a.shape)
```

### 获取矩阵的大小

```python
a = np.arange(15).reshape(3, 5)
# 获取矩阵的大小
    print(a.shape)
```

### 获取矩阵的维度

```python
a = np.arange(15).reshape(3, 5)
获取矩阵维度   一维数组 二维等
print(a.ndim)
```

### 获取数组元素的类型

```python
 # 获取数组中元素的类型
    print("获取元素类型", a.dtype.name)
```

### 获取数组中元素的大小

```python
    # 获取数组中元素的大小
    print("获取数组中元素的大小", a.itemsize)
```

### 获取元素总个数

表示数组中元素的总数，等同于ndarray.shape中两个元素的乘积

```python
    # 获取元素总数量
    print("获取元素总数量", a.size)
```

### 类型

```python
type(a)
<type 'numpy.ndarray'>
```

# 基础使用

## 创建Numpy数组

### array方法

```python
import numpy as np
    a = np.array([2, 3, 4])  # 创建一维数组
    b = np.array([(2, 3, 4), (5, 6, 7)])  # 创建二维数组
    c = np.array([[1, 2], [3, 4]], dtype=np.float64)  # 创建指定数据类型的数
```

如果想创建指定shape的数组，并使用占位符来初始化数组，可以用以下方法：

```python
a = np.array([2, 3, 4])  # 创建一维数组

    b = np.array([(2, 3, 4), (5, 6, 7)])  # 创建二维数组
    
    c = np.array([[1, 2], [3, 4]], dtype=np.float64)  # 创建指定数据类型的数
    
    d = np.zeros((3, 4))  # 创建3行4列矩阵，用0初始化矩阵中所有的元素
    
    e = np.ones((2, 3, 4), dtype=np.int16)  # 创建三维矩阵，维度分别为2，3，4，且用1来初始化矩阵中所有的元素
    
    f = np.empty((2, 3))  # 创建2行3列空矩阵，矩阵中元素初始值随机，取决于内存状态，默认情况下，创建的数组的dtype为float64
    g = np.arange(6)    
    
    h = np.arange(10, 30, 5)
    
    i = np.full((4, 3), 7)  # 创建4行3列元素都是7的矩阵
    
    j = np.eye(3)  # 创建3行3列的单位矩阵

```

对应的`zeros、ones、empty`还有`zeros_like、ones_like、empty_like`，它们以另一个数组为参数，根据其形状和dtype创建数组。

```python
np.zeros( (3,4) )
#创建3行4列矩阵，用0初始化矩阵中所有的元素

np.ones( (2,3,4), dtype=np.int16 ) 
#创建三维矩阵，维度分别为2，3，4，且用1来初始化矩阵中所有的元素

np.empty( (2,3) ) 
#创建2行3列空矩阵，矩阵中元素初始值随机，取决于内存状态，默认情况下，创建的数组的dtype为float64。

```

对应的`zeros、ones、empty`还有`zeros_like、ones_like、empty_like`，它们以另一个数组为参数，根据其形状和dtype创建数组。

### arange方法，与Python内置的range相似

```python
    
   g = np.arange(6)
    
    h = np.arange(10, 30, 5)
    
    print(type(h))
    
    print(h)
```

### reshape()函数

```
np.arange(6).reshape(1,-1)
```



## 数组基本数学操作

### 加减操作

```python
 a = np.array([[1, 2], [3, 4]])
    b = np.array([[5, 6], [7, 8]])
    print(a)
    # a+b
    print("a+b对应位置相加\n", a + b)
    # a-b
    print("a-b对应位置相减\n", a - b)
    # a*b
    print("a*b 相乘\n", a * b)
    # a/b
    print("a/b 相除\n", a / b)
    # 对数组a开平方
    print("a对应位置开平方\n", np.sqrt(a))
    # 矩阵点乘
    print("矩阵点乘\n", a.dot(b))
    # 矩阵点乘
    print("矩阵点乘\n", np.dot(a, b))
    # 最大值
    print("最大值\n", a.max())
    # 最小
    print("最小值\n", a.min())
    # 求和
    print("求和\n", a.sum())
```



## 常用函数

```python
    a = np.array([[1, 2], [3, 4], [5, 6]])
```

### 函数sum有相求和功能

```python
    # 将数组a中所有元素的总和
    print("将数组a中所有元素的总和", np.sum(a))
    # 将数组a中行与列进行求和
    print("将数组a中行与列进行求和",np.sum(a,axis=0))
    # 将数组a中行与行进行求和
    print("将数组a中行与行进行求和", np.sum(a, axis=1))
```

### 函数mean有求平均值的功能

```python
    # 求数组a中所有元素的平均值
    print("求数组a中所有元素的平均值", np.mean(a))
    # 求数组a中行与列的平均值
    print("将数组a中行与列进行求和", np.mean(a, axis=0))
    # 求数组a中行与行的平均值
    print("将数组a中行与行进行求和", np.mean(a, axis=1))

```

### 函数uniform有产生指定范围数值的功能

```python
 # 在1-4之间随机产生指定范围数值的功能
    print("在1-4之间随机产生指定范围数值的功能",np.random.uniform(1,4))
```

### 函数tile有产生指定范围数值的功能 

类似扩列功能

```python
  print("========函数tile有产生指定范围数值的功能================")

    print("在横向为2增加一个数组a,纵向1不增加", np.tile(a, (2, 1)))
    print("在横向为3增加2个数组a,纵向2增加1个数组", np.tile(a, (3, 2)))
```

