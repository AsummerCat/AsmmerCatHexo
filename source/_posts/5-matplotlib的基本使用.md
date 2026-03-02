---
title: 5.matplotlib的基本使用
date: 2026-03-02 11:51:19
tags: [机器学习]
---

# matplotlib的基本使用

这是一个绘图库

## 1.1  matplotlib.pyplot 模块

matplotlib.pyplot 模块包含了一系列类似于matlab的画图函数

<!--more-->

```
import matplotlib.pyplot as plt

```

## 1.2 创建画布
```

    plt.figure(figsize(200*200),dpi=200)

    plt.figure()
```

figsize :指定图的长宽
dpi: 图像的清晰度
返回fig对象

## 1.3 绘制图像
```

    plt.plot(x,y)
    #以折线图为例
    # x轴 y轴
```



## 1.4 显示图像

    plt.show() 

## 1.5 新增辅助线/网格

```
plt.grid(True,linestyle='--',alpha=0.5)

```

True=是否显示
alpha=透明度

## 1.6 保存图像
```

    plt.savefig("./test.png")
```

保存图像到指定位置
注意: plt.show()会释放figure资源,如果在显示图像之后保存图像的话,会出现空图片

## 1.7 在一个画布上绘制多个线条
```

    # 绘制多条线
    plt.plot([1, 2, 3, 4])
    # 绘制另外一条线 并且 赋予红色 机器虚线
    plt.plot([5, 6, 7, 8],color='r',linestyle='--')
```

## 1.8 添加图例 也就是图像上显示tab的小格子 plt.legend
```

    plt.figure()
    # 绘制多条线
    plt.plot([1, 2, 3, 4],label='上海')
    # 绘制另外一条线 并且 赋予红色 机器虚线 ,显示label
    plt.plot([5, 6, 7, 8],color='r',linestyle='--',label='北京')
    # 显示小格子 可以使用数字 也可用英文的居左居右
    # plt.legend(loc=1)
    plt.legend(loc="best")
```

## 1.9 多个坐标系的绘图 也就是一个画布上 可以出现多个完整的图例

# 创建画布
```
fig,axes =plt.subplots(nrows=1,ncols=2 ,figsize=(20,8),dpi=100)

    # 进行图表的展示
    import matplotlib.pyplot as plt
    import random
    # 设置解决中文字体问题
    from matplotlib import rcParams
    rcParams['font.family'] = 'Microsoft YaHei'

    # 绘制多个画布
    x=range(60)
    y_shanghai=[random.uniform(15,18) for i in x]
    y_beijing=[random.uniform(1,5) for i in x]

    # 创建画布
    fig,axes =plt.subplots(nrows=1,ncols=2 ,figsize=(20,8),dpi=100)

    # 绘制图像
    axes[0].plot(x,y_shanghai,label="上海")
    axes[0].legend(loc="best")
    axes[1].plot(x,y_beijing,color='r',linestyle='--',label='北京')
    axes[1].legend(loc="best")
    fig.show()
```

## 案例

```
# 进行图表的展示
import matplotlib.pyplot as plt
%matplotlib inline

# 设置解决中文字体问题
from matplotlib import rcParams
rcParams['font.family'] = 'Microsoft YaHei'

# 绘制画布
plt.figure()
# 绘制多条线
plt.plot([1, 2, 3, 4],label='上海')
# 绘制另外一条线 并且 赋予红色 机器虚线 ,显示label
plt.plot([5, 6, 7, 8],color='r',linestyle='--',label='北京')
# 显示小格子
# plt.legend(loc=1)
plt.legend(loc="best")
plt.ylabel('some numbers')
plt.xlabel('测试')
plt.grid(True,linestyle='--',alpha=0.5)
plt.show()

```

