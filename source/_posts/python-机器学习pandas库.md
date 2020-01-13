---
title: python-机器学习pandas库
date: 2020-01-09 22:46:45
tags: [python,机器学习]
---

# python-机器学习pandas库

数据模型  是一个数据分析库

官网: http://pandas.pydata.org

##  [demo地址](https://github.com/AsummerCat/pandas_demo)



## 引入模块

```python
import numpy as np
import pandas as pd
```

<!--more-->

## 基础数据结构 dataFrame

```python
# -*- coding: utf-8 -*-

import numpy as np
import pandas as pd


'''
表格型基础数据结构
'''
def test():
    # 基本数据结构 Data Structure
    s = pd.Series([i * 2 for i in range(1, 11)])
    print(type(s))
    # 随机数据
    dates = pd.date_range("20170301", periods=8)

    # DataFrame 数据结构
    ## 定义方式一
    ###  index表示主键  columns属性值
    df = pd.DataFrame(np.random.randn(8, 5), index=dates, columns=list("ABCDE"))
    print(df)
    ## 定义方式二
    df1 = pd.DataFrame({"A": 1,
                        "B": pd.Timestamp("20170301"),
                        "C": pd.Series(1, index=list(range(4)), dtype="float32"),
                        "D": np.array([3] * 4, dtype="float32"),
                        "E": pd.Categorical(["police", "student", "teacher", "doctor"])})
    print(df1)

    pass


if __name__ == '__main__':
    test()

```

## 基本操作

### 了解属性的大致

```python

    ## 打印前几行
    print(df.head(3))

    ## 打印后几行
    print(df.tail(3))

    ## 打印主键
    print(df.index)

    ## 打印值
    print(df.values)

    ## 横纵列转换格式
    print(df.T)

    ## 查看 根据某一列排序 降序
    print(df.sort_values("C"))

    ## 根据index 进行排序  并且禁止降序处理
    print(df.sort_index(axis=1, ascending=False))

    ## 大致了解数据   求出所有属性值的 最大值 最小值 平均值
    print(df.describe())

    print("=" * 100)
```

### 选择数据 切片操作

```python
'''
    选择数据  切片
    '''
    # 直接打印A的属性列
    print(df["A"])

    # 获取0-3行的数据 切片
    print(df[:3])

    # 获取1号-4号的数据 切片
    print(df["20170301":"20170304"])

    # 提取指定主键的数据
    print(df.loc[dates[0]])
    # 获取指定范围数据 并且提取指定属性
    print(df.loc["20170301":"20170304", ["B", "D"]])

    # 根据主键  ->获取指定的属性
    print(df.at[dates[0], "C"])

    # 根据下标 获取指定行数数据
    print(df.iloc[1:3, 2:4])
    # 根据下标 获取第0行第四列的值
    print(df.iloc[0, 4])
    # 跟上面类似 获取指定位置的值
    print(df.iat[0, 4])
```

### 筛选数据

```python
 # 筛选数据
    ## 筛选符合 B>0 和A<0的记录
    print(df[df.B > 0][df.A < 0])
    ## 筛选 df内所有值>0的 不符合为NaN
    print(df[df > 0])

    ## 筛选 E属性存在某个几个值的 类似 数据库的in
    print(df[df["E"].isin([1, 2])])
```

### 赋值 dataFrame

```python
    '''
    dataFrame赋值
    '''
    # 创建一个新的数据项
    s1 = pd.Series(list(range(10, 18)), index=pd.date_range("20170301", periods=8))
    ## 赋值  根据主键将数据赋值给F
    df["F"] = s1

    ## at 根据指定位置赋值
    df.at[dates[0], "A"] = 0

    ## iat  在数据网格的1,1的位置   进行修改赋值
    df.iat[1, 1] = 1
    ## loc 选择属性 替换所有 直接赋值
    df.loc[:, "D"] = 1
```

###  拷贝

```python
    '''
    拷贝一份dataFrame
    '''
    df2 = df.copy()
    ## 所有正数修改为负数
    df2[df2 > 0] = -df2
    print(df2)
```

### 删除列

```python
# 创建一个新的数据项
    s1 = pd.Series(list(range(10, 18)), index=pd.date_range("20170301", periods=8))
    
# 删除列
del df['A']
# 查询列名
print(df.columns)

#丢弃指定轴上的项
s1.drop('C')
s1.drop(['C','D'])

df = pd.DataFrame(np.random.randn(8, 5), index=dates, columns=list(["Colorado","Ohio"]))
# 对于dataFrame 可以删除任意轴上的索引值
print(df1.drop([dates[0], dates[2]]))

```

### 科学计算

```python
    '''
    科学计算
    '''
    # 求平均值
    print(df.mean())
    # 方差
    print(df.var())

    s = pd.Series([1, 2, 4, np.nan, 5, 7, 9, 10], index=dates)
    print(s)
    # 所有的值移后两位 后面的值不会移动至前面
    print(s.shift(2))

    # 阶分 填入数值表示多阶 后面一个减去前面一个
    print(s.diff())

    # 每个值在series出现的次数
    print(s.value_counts())

    print(df)
    # 累加 后面的值都是前面的累加值
    print(df.apply(np.cumsum))

    # 自定义  极差
    print(df.apply(lambda x: x.max() - x.min()))
```



## 缺失值处理

```python
   '''
    缺失值处理
    '''
    ## 获取原数据的前4行 获取属性 ABCD 新增G属性
    df1 = df.reindex(index=dates[:4], columns=list("ABCD") + ["G"])
    ## 仅给第一行第二行 赋值
    df1.loc[dates[0]:dates[1], "G"] = 1
    print(df1)
    
   
   '''
   缺失值
   '''
   # series的reindex将会根据新索引进行重排,吐过某个索引值不当前不存在,就就引入缺失值
   obj.reindex(['a','b','c','d','e'].fill_value=0)

    ## 缺失值处理方式
    ### 一  ->直接丢弃
    print(df1.dropna())
    ### 二 ->赋值一个指定值 或者插值
    print(df1.fillna(value=2))
```

### 检测缺失数据 判断是否为空

```python
 # 列表返回true false
 pd.isnull(df)
 
 # 列表返回false true
 pd.notnull(df)
```



## 表的拼接和重塑

###  拼接

###  dataFrame相加   df1+df2  

```python
df1+df2
将他们相加时,没有重叠的位置会产生Na值
```

![相加](/img/2020-01-13/1.png)

### 也可以使用 df1.add方法

```python
df1.add(df2,fill_value=0)
```

![相加2](/img/2020-01-13/2.png)

```python
  '''
    拼接
    '''
    # 获取前三行后三行
    pieces = [df[:3], df[-3:]]
    # 拼接
    print(pd.concat(pieces))

    left = pd.DataFrame({"key": ["x", "y"], "value": [1, 2]})
    right = pd.DataFrame({"key": ["x", "z"], "value": [3, 4]})

    #   合并  类似数据库的left join   对比两个DataFrame  on=对比的key how=默认inner  left right outer(全部显示)
    print(pd.merge(left, right, on="key", how="left"))

    df3 = pd.DataFrame({"A": ["a", "b", "c", "b"], "B": list(range(4))})

    # 聚合函数 类似数据库的group by 
    print(df3.groupby("A").sum())
```

![灵活的算术方法](/img/2020-01-13/3.png)

###   重塑 Reshape

重塑 透视表 交叉分析

```python
 '''
    重塑 透视表->交叉分析
    '''
    import datetime
    df4 = pd.DataFrame({"A": ['one', 'one', 'two', 'three'] * 6,
                        "B": ['a', 'b', 'c'] * 8,
                        "C": ['foo', 'foo', 'foo', 'bar', 'bar', 'bar'] * 4,
                        "D": np.random.randn(24),
                        # 状态分布
                        "E": np.random.randn(24),
                        "F": [datetime.datetime(2017, i, 1) for i in range(1, 13)] +
                             [datetime.datetime(2017, i, 15) for i in range(1, 13)]})

    # 透视表
    ## 输出值:D 主键:AB 字段:C
    print(pd.pivot_table(df4, values="D", index=["A", "B"], columns=["C"]))
```



## 时间、绘图、文件操作

### 时间序列 time series

```python
    '''
    时间序列
    time series
    '''
      # 定义时间序列 periods=时间段 feq:时间格式 Y M H D S
    t_exam = pd.date_range("20170301", periods=10, freq="S")
    print(t_exam)
```

### 绘图  Graph

```python
    '''
    绘图功能 
    Graph
    '''
    # 创建序列
    ts = pd.Series(np.random.randn(1000), index=pd.date_range("20170301", periods=1000))
    # 累加
    ts = ts.cumsum()
    # 需要引入  from pylab import *
    ts.plot()
    show()
```

## 文件操作

```python
    '''
    文件操作
    '''
    # 读取csv文件
    df6 = pd.read_csv("./test.csv")
    print(df6)
    # 读取xlsx文件 第一块
    df7 = pd.read_excel("./test.xlsx", "Sheet1")
    print(df7)

    # 保存csv
    df6.to_csv("/test2.csv")
    # 保存xlsx
    df7.to_excel("/test2.xlsx")
```

