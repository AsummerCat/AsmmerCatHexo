---
title: 6.Numpy的基本使用
date: 2026-03-02 11:53:20
tags: [机器学习]
---

```
科学计算 矩阵 之类的
NumPy 是一个为 Python 带来数组对象和操作函数的库，是进行科学计算的基础包。

```

## 2.基础方法

    import numpy as np

## 2.1 数组 ndarray
```

np.array

    import numpy as np
    score=np.array(
        [[1,2,3,4],
        [2,3,4,5],
        [5,6,7,8]]
    )
    print(score)
```

<!--more-->


#### 2.1.1 ndarry的属性
```

    array.shape      数组维度的元组
    array.ndim       数组维数
    array.size       数组中的元素数量
    array.itemsize   一个数组元素的长度(字节)
    array.dtype      数组元素的类型
```

#### 2.1.1 ndarry的形状
```

    import numpy as np

    a=np.array=([1,2,3],[4,5,6])
    b=np.array=([1,2,3])
    c=np.array=([[[1,2,3],[4,5,6]],[[1,2,3],[4,5,6]]])

    print(a)  # 二维数组
    print(b)  # 一维数组
    print(c)  # 三维数组
```

## 2.2 生成数组的方法

#### 2.2.1 生成0和1的数组
```

    import numpy as np
    # 生成1的数组
    ones=np.ones([4,8])
    # 拷贝1的数组
    ones2=np.ones_like(ones)
    print(ones)

    # 生成0的数组
    zeros=np.zeros([4,8])
    # 拷贝0的数组
    zeros2=np.zeros_like(zeros)
    zeros
```

#### 2.2.2 从现有数组中生成

np.copy 拷贝数组
np.asanyarray 拷贝数组(浅拷贝)
```

    import numpy as np

    # 正确的方式定义数组
    a = np.array=([[1,2,3],[4,5,6]])
    # 拷贝数组
    b = np.copy(a)

    print(a)
    print("-------------")
    print(b)
```

#### 2.2.3 生成固定范围的数组

###### 2.2.3.1 等差数组 np.linspace

np.linspace(start,stop,num,endpoint)
创建等差数组 -指定数量
参数:
start: 序列起始值
stop: 序列的终止值
num: 要生成的等间隔样本数量,默认50
endpoint: 序列中是否包含stop值 默认为true

    # 生成等间隔的数组
    a=np.linspace(0,100,11)
    print(a)

###### 2.2.3.2 等差数组 np.arange

np.arange(start,stop,num,dtype)
创建等差数组 -指定步长
参数:
stop: 步长 默认为1

    a=np.range(10,50,2)
    print(a)

###### 2.2.3.2 生成等比数列 np.logspace

np.arange(start,stop,num,dtype)
创建等比数列
参数:
num: 要生成的等比数列数量 ,默认为50

    # 生成等比数列 计算是 10的 0次方 ->10的2次方 之间均分布获取三个值
    c=np.logspace(0,2,3)
    print(c)

#### 2.2.4 生成随机数组   np.random模块

正态分布: u是服从正态分布的随机变量的均值,第二个参数@ 是由此随机变量的反差, 所以正态分布记作N(u,@)

u决定了其位置,其标准差@决定了分布的幅度, 当u=0,@=a时的正态分布是标准正态分布

标准差是->反差得来的

np.random.randn(d0,d1,...,dn)
:从标准正态分布中返回一个或者多个样本值

##### 2.2.4.1 正态分布

np.random.normal(loc=0.0,scale=1.0,size=None)
loc:此概率分布的均值(对应着整个分布的中心center)
scale: 此概率分布的标准差 (对应于分布的宽度,scale越大越矮胖,scale越小,越瘦高)
size: 输出的shape  默认为None ,只输出一个值

np.random.standard\_noraml(size=None)
:返回指定形状的标准正态分布的数组
```

    # 生成随机数组
    import numpy as np
    # 生成均值为1.75,标准差为1的正态分布数据,10000个
    x1 =np.random.normal(1.75,1,1000)
    print(x1)

    # 比如随机创建一个4行5列 ,某正态分布内,比如均值为0,方差为1
    ## 股票涨跌幅度
    stock_change=np.random.normal(0,1,[4,5])
    print(stock_change)
```

##### 2.2.4.1 均匀分布
```

np.random.rand(d0,d1,...,dn)
返回\[0.0->1.0]内的一组均匀分布的数

返回处于low 和high 之间的若干个均匀分布的数
np.random.uniform(low=0.0,high=1.0,size=None)
low: 最小范围
high: 最大范围
size: 输出的数量

返回处于low 和high 之间的若干个均匀分布的整数
c=np.random.randint(low=0,high=6,size=None)
```

案例:
```

    # 生成随机数组
    import numpy as np
    ## 返回[0.0->1.0]内的一组均匀分布的数
    a=np.random.rand()
    print(a)

    ## 返回处于low 和high 之间的若干个均匀分布的数
    b=np.random.uniform(low=0.0,high=6.0,size=None)
    print(b)

    ## 返回处于low 和high 之间的若干个均匀分布的整数
    c=np.random.randint(low=0,high=6,size=None)
    print(c)
```

## 3.数组的基本操作

#### 3.1 数据的索引和切片

获取索引方式:
直接进行索引,切片
对象\[:,0:3]--先行后列

二维数组数组获取索引方式:

    # 第一行的 前三列
    stock_change[0,0:3]

三维数组数组获取索引方式:

    # 第一行的第二个值
    a1[0,0,1]

#### 3.2 形状改变

`ndarray.reshape(shape,order)`
返回一个具有相同数据域,但shape不一样的视图
行和列 不进行转换

将数据押平后进行排序,产生一个新的对象

```
## 形状改变 转换为2行5列的数据
print(a.reshape([2,5]))

```

resize 改变数组本身的形状
会修改原来a里的数组形状

```

print(a.resize([1,5]))
```

将行列进行互换 .T.shape
`T.shape`

    ## 将行列进行互换
    print(a.T.shape)

#### 3.3 类型修改 astype

`a.astype(type)`

    ## 类型修改
    a.astype(np.int16)

#### 3.4数组去重 np.unique
```

    import numpy as np

    # 先获取7个随机数
    a=np.random.uniform(low=0,high=6,size=10)

    # 数组去重
    np.unique(a)
```

## 4.ndarray运算

#### 4.1 逻辑运算

```
a=np.random.uniform(low=0,high=6,size=10)

#ndarray运算
a=a.astype(np.int16)
print(a)
# 逻辑判断
##原值:[3 1 1 4 1 0 4 2 1 3]
##输出:[ True False False  True False False  True False False  True]
print(a>2)

# BOOL赋值
test_score[test_score>60]=1
## 会改变符合条件的值为1 并且输出test_score ,不符合的值 则不变

```

#### 4.2 判断函数

`np.all()` :判断 范围内容的数据是否都为true
```

    np.all(score[0:2,:]>60)
```

    返回false

`np.any()` :判断 范围内容的数据是有为true
```

    np.any(score[0:2,:]>60)
```

    如果存在一个符合,则返回true

#### 4.3 三元运算符

np.where

如果存在复合逻辑可用 `np.logical_and` 和 `np.logical_or`

```
np.where(temp>60,1,0)

表示如果temp中大于60的值 输出为1,其他为0
## 输出为:array([99,  0,  0,  0,  0,  0,  0, 99, 99, 99])

np.where(np.logical_and(a>1,a<3),1,0)
## 输出为:array([0, 0, 0, 0, 0, 0, 0, 0, 0, 1])

```

#### 4.4 统计运算

```
axis=0 代表按行处理
axis=1 代表按列处理

min(a,axis): 最小值

max() :最大值

median() : 中位数

mean(): 平均值

std(): 波动情况

var(): 方差  

argmax(): 最大值的数据下标

argmin(): 最小值的数据下标

```

#### 4.5 数组间的运算

直接对于数组进行

```

arr+1 对所有元素进行加一

arr/2 对所有元素进行除以2

arr*3 对所有元素进行复制三份  比如[3,3] 变为[9,9]

```

#### 4.6 转换为矢量图 (数组计算的前提 需要转换成一样的shape)

```
arr1=np.array([[0],[1],[2],[3]])
arr1.shape
# 转换为 (4,1)  4行1列

```

#### 4.7 数组相加

比如满足条件 某一维度等长 ,或者其中一组数组的某一维度为一

```
arr1=np.array([[0],[1],[2],[3]])
arr1.shape

arr2=np.array([0],[1],[2],[3])
arr2.shape

arr1+arr2

##这样会对arr1和arr2进行扩展 都变为 4行3列的数据
每个元素位相加

简单理解就是:
arr1 进行对4行3列平铺  也就是 每行的数据都是扩展相同的数据出来的
arr2 对arr1的平铺的结果也进行平铺 并且进行相加

```

## 5.矩阵

3\*2的矩阵 也就是3行2列

#### 5.1向量

向量是一种特殊的矩阵, 向量(3\*1) 表示

#### 5.2 矩阵的加法

矩阵的加法: 行列数相等的可以加

矩阵的乘法: 每个元素都要乘

#### 5.3 矩阵向量乘法

M*N的矩阵乘以N*1的向量 ,得到的是M\*1的向量

#### 5.4 矩阵乘法

M*N矩阵乘以N*O的矩阵,变成M\*O矩阵

#### 5.5 矩阵计算
```
//矩阵*矩阵
np.matmul(a,b)
//矩阵*矩阵 也可以乘以标量
np.dot(a,b)
```

