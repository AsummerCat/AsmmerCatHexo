---
title: 8.使用seaborn更加高效的绘图
date: 2026-03-02 11:57:38
tags: [机器学习]
---

# 1.安装依赖

    pip3 install seaborn

    或者
    conda install seaborn

<!--more-->

## 2.导入使用

    import seaborn as sns

# 3.绘制单变量分布 distplot

seaborn.distplot(a, bins=None, hist=True, kde=True, rug=False, fit=None, color=None)
参数:
a:表示要观察的数据 ,可以是Series,一维数组和列表
bin: 用于控制条形的数量
hist:  是否绘制(标注)直方图
kde :表示是否绘制高斯核密度估计曲线
rug: 是否支持在轴方向上绘制rugplot

    import numpy as np
    import seaborn as sns

    np.random.seed(0)
    arr =np.random.rand(10)

    print(arr)

    # 绘制直方图
    ax=sns.displot(arr,bins=10,kde=True,rug=True)

# 4.绘制双变量分布 jointplot

seaborn.joinplot(x,y,data=None,
kind='scatter' ,stat\_func=None ,color='r' , ratio=5, space=0.2 ,dropna=True)
参数:
kind: 表示绘制图形的类型
stat\_func: 用于计算有关关系的统计量并标注图
color: 表示绘图元素的颜色
size: 用于设置图的大小(正方形)
ratio: 表示中心图和侧边图的比例, 参数越大,中心图的占比越大
space: 用于设置中心图与侧边图的间隔大小

*   绘制散点图

```
import numpy as np
import pandas as pd
import seaborn as sns

# 创建DataFrame对象
dataframe_obj=pd.DataFrame({"x":np.random.randn(500),"y":np.random.randn(500)})

## 绘制散点图
sns.jointplot(data=dataframe_obj,kind='scatter')

```

*   绘制直方图

```
# 绘制直方图
import numpy as np
import pandas as pd
import seaborn as sns
# 创建DataFrame对象
dataframe_obj=pd.DataFrame({"x":np.random.randn(500),"y":np.random.randn(500)})

# 绘制直方图
sns.jointplot(x="x",y="y",data=dataframe_obj,kind="hex")
## 绘制核密度直方图
sns.jointplot(x="x",y="y",data=dataframe_obj,kind="kde")


```

*   绘制成对的双变量分布  (显示每对变量的关系)

```
# 绘制双变量分布
import numpy as np
import pandas as pd
import seaborn as sns

# 加载seborn中内置的数据集
dataset=sns.load_dataset("iris")
#dataset.head()

# 绘制多个成对的双变量分布
sns.pairplot(dataset)

```

