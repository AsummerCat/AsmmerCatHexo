---
title: 7.Pandas的基本使用
date: 2026-03-02 11:54:47
tags: [机器学习]
---

# 1.Pandas的数据结构

*   增强图表可读性
*   便捷的数据处理能力
*   读取文件方便
*   封装了 Matplotlib ,Numpy的画图和计算

`series` 一维数据结构
`DataFrame1`和`DataFrame2` 二维数据结构
`multiindex` 三维数据结构(比较少用到)

<!--more-->


### 1.1 Series的创建

`pd.Series(data=None,index=None,dtype=None)`
参数:
data: 传入的数据,可以是ndarray ,list等
index: 索引,必须是唯一的,并与数据长度相等, 如果没有传入默认自动创建一个0-N的整数索引
dtype: 数据的类型

*   指定内容,默认索引

<!---->

    import pandas as pd
    import numpy as np
    # pd.Series(data=None,index=None,dtype=None)

    pd.Series(np.arange(10))

*   指定索引

<!---->

    import pandas as pd
    pd.Series([1,2,3,4,5,6,7],index=[1,2,3,4,5,6,7])

*   通过字典数据创建

<!---->

    import pandas as pd
    ## 通过字典数据创建
    pd.Series({'red':100,'blue':200,'green':300,'yellow':1000})

*   根据下标获取值

<!---->

    a[1]

### 1.2 DataFrame的创建

类似二维数组或者excel的对象, 又有行索引,又有列索引

`pd.DataFrame(data=None,index=None,columns=None)`
参数:
ps:(这里类似于excel的表头 X,Y轴显示)
index: 行标签,如果没有传入默认自动创建一个0-N的整数索引\
columns: 列标签 ,如果没有传入默认自动创建一个0-N的整数索引

*   指定内容

```
# DataFrame的创建
import pandas as pd
import numpy as np

## 通过已有数据进行创建
score=np.random.randint(40,100,(10,5));
score_df=pd.DataFrame(score)

subjects =['语文','英语','法文','韩语','日语']
stu=['同学'+str(i) for i in range(score_df.shape[0])]
data=pd.DataFrame(score,columns=subjects,index=stu)
print(data)


## 


```

*   属性

<!---->

    data.shape

    输出:显示有多少行和列

<!---->

    data.index

    获取DataFrame的行索引列表

<!---->

    data.columns

    获取DataFrame的列索引列表

<!---->

    data.values

    获取DataFrame的的array的值

<!---->

    data.T

    可以将行列进行转换

<!---->

    data.head(5)

    只显示前某几行数据 ,默认5行

<!---->

    data.tail(5)

    只显示后某几行数据 ,默认5行

*   索引的修改

<!---->

    stu=['同学__'+str(i) for i in range(score_df.shape[0])]
    data.index =stu

*   重建索引

```
stu=['同学__'+str(i) for i in range(score_df.shape[0])]
data.index =stu

data.reset_index(drop=False) 

# 如果为Ture 则表示删除原来的索引值,重建

```

*   设置某一列为新的索引

<!---->

    data.set_index('列名')

# 2.Pandas的基础使用

#### 2.1 读取文件 read\_csv

    import pandas as pd

    # 读取文件
    data=pd.read_csv("./day.csv")

    # 删除一些列 
    data=data.drop(["md","month"],axis=1)

#### 2.2 索引操作

*   直接使用行列索引(先列后行)

<!---->

    data['open']['index的名称']

    输出open这个列在 index这个位置的数据

*   结合loc 或者iloc使用索引

loc只能指定行列索引的名字

```
data.loc['2018':'2019','open']
获取2018->2019 之间的index open列的数据

```

*   iloc 根据index的坐标去获取数据

<!---->

    data.iloc[:3,:5]
    获取前三个index的 前五列的数据

*   使用ix组合索引

<!---->

    获取.ix 前四列的数据

    data.ix[0:4,['open','close','high','low']]

    # 推荐使用 loc和iloc来获取

    data.loc[data.index[0:4],['open','close','high','low']]

*   data.head() 默认获取前五行

#### 2.3 赋值操作

```
#将数据列close的所有数据修改为1
data['close']=1
或者
data.close=1

# 将指定index的close字段修改为1 先列后行
data['close']['index名称']=1

```

#### 2.3 排序  //根据索引排序 或者 根据内容进行排序

内容进行排序

    语法: df.sort_values(by= ,ascending=)
    参数:
      by: 指定排序参考的键
      ascending: 默认升序  True:升序, False: 降序

例如:

    data.sort_values(by="open",ascending=True)

    按照多个键进行排序
    data.sort_values(by=['open','close'],ascending=True)

根据索引进行排序

    data.sort_index()

*   Series的排序

<!---->

    data['字段名称'].sort_values(ascending=True)

#### 2.4 DataFrame运算

*   算术运算

<!---->

    加法:
    data['open'].add(1)
    减法:
    data['open'].sub(1)
    乘法: 
    mul()
    除法: 
    div()

    给open的列 加减1

*   逻辑运算

<!---->

     data['open']>23 
    返回逻辑结果 open的列会变为 True 和False

     data[data['open']>23 ].head()
    返回符合条件的前几列数据 

     data[(data['open']>23) & (data['open']<25 ) ].head()
     open列大于23 并且 小于25

逻辑运算函数

*   query(expr) 查询语句

<!---->

    data.query("open<24 & open>23").head()

*   isin()

<!---->

    data[data['open']].isin([23,24])
    # 用来筛选出符合条件的数据

*   统计运算
    `describe函数`

<!---->

    data.describe()

    # 直接计算出 平均值,标准差,最大值,最小值



    # 单独计算的统计函数
    默认对列进行计算
     0 代表列求结果
     1 代表行求结果
     
    data.max(0)
    data.var(0)

\*\* 累计统计函数

    data['open'].cumsum() 计算N个数的和

    data['open'].cummax() 计算N个数的最大值

    data['open'].cummin() 计算N个数的最小值

    data['open'].cumprod() 计算N个数的积

\*\* 自定义运算

    data[['open','close']].apply(lambda x: x.max()-x.min(),axis=0)


    函数: apply(表达式,axis)

*   画图
    plot(kind='line')
    默认 折线图
    可以传入参数 定义图形类型
    'line' 'bar' ,'pie'

```
import matplotlib.pyplot as plt

data.plot()

```

# 3.Pandas的高级使用

#### 3.1 缺失值处理

isnull 判断是否有缺失数据NaN
fillna 缺失值填充
dropna 缺失值删除
replace 数据替换

    判断数据是否有缺失值

    # 是否存在
    pd.isnull(df)
    # 是否不存在
    pd.notnull(df)

    使用np.all(pd.notnull(df)) 进行组合判断  如果存在一个缺失值就返回false
    使用np.any(pd.isnull(df)) 进行组合判断  如果存在一个缺失值就返回True

<!---->

    # 删除缺失值
    b=df.dropna(axis='rows')

<!---->

    c=df.fillna(value,inplace=True)

    注意以上两个方法都要返回新对象,不会修改到原来的对象

#### 3.2 数据离散化

使用 cut ,qcut 实现数据的区间分组 (主要是为了数据离散化 用来减少连续属性的数据个数)
使用 get\_dummies 实现数据的one-hot编码

    # 自动分组 10表示分为10组
    qcut=pd.qcut(data,10)

    # 计算分到每个组的数据个数
    qcut.value_counts()

    # 自定义区间分组
    ## 指定区间
    bins=[-100,-7,-5]]
    p_counts=pd.cut(data,bin)

*   one-hot编码

<!---->

    入参 分组后的数据
    dummies=pd.get_dummies(p_counts,prefix=None)

#### 3.3 合并

使用 pd.concat实现数据的合并
使用pd.merge实现数据的合并

*   数据连接 pd.concat

<!---->

    axis=0 为列 
    axis=1 为行
    按照行索引或者列索引进行合并

    pd.concat([data,dummies],axis=1)

*   数据合并 pd.merge

```
可以按照两组数据进行合并

# pd.merge(left,right,how='inner',on=None)
参数: left A表
      right B表
      on: 指定共同建
      how 根据什么方式进行连接
       left right outer inner 四种连接方式 默认`inner`

result=pd.merge(left,right,on=['key1','key2']) 


```

#### 3.4 交叉表和透视表

使用`pd.crosstab` 交叉表
使用`pd.pivot_table`透视表

*   交叉表 pd.crosstab
    用来计算一列数据对另外一列数据的分组个数

<!---->

    # 找寻两列数据之间的关系
    pd.crosstab(data['open'],data['close'])

*   透视表 pd.pivot\_table
*   指定某一列数据和另外一列数据的关系 百分比

<!---->

    data.pivot_table('A','B')

#### 3.5 分组和聚合

使用`groupby`函数 进行来分组和聚合

data.groupby(key,as\_index=False)

ps: as\_index 表示是否需要显示原来的索引

```
# 分组,求平均值
## 根据color进行分组后,对price1字段进行求平均值
data.gorupby('color')[price1].mean()


```

