---
title: python-机器学习matplotlib库
date: 2020-01-08 00:07:49
tags: [python,机器学习]
---

# matplotlib库

## 用来绘制图表 绘图库

Python第三方库Matplotlib，讲解绘制坐标系、散点图、极坐标图等直观展示数据趋势和特点的方法；

<!--more-->

## 官网 ->matplotlib.org

## [demo地址](https://github.com/AsummerCat/matplotlib_demo)

## 导入模块

```python
import matplotlib.pyplot as plt
import numpy as np
```

<!--more-->

## 方法

### 标题

```python
plt.title("cos&sin")
```

### 网格线

```python
plt.grid()
```

### 设置横轴纵轴标记点显示

```python
  # 设置横轴 横轴标示的位置  标示的内容
    plt.xticks([-np.pi, -np.pi / 2, 0, np.pi / 2, np.pi], [r'$-\pi$', r'$-\pi/2$', r'$0$', '$+\pi/2$', '$+\pi$'])
    # 设置纵轴 -1,1 5个点 并且显示
    plt.yticks(np.linspace(-1, 1, 5, endpoint=True))
```

### 图例图

```python
# 图例图 就是左边有个标记样式的
    plt.legend(loc='upper left')
```

### 标记点 添加注释

```python
# 标记点 添加注释    注释名称 点的位置   标记的偏移量 x+10 y+30   相对位置 箭头类型
    plt.annotate("cos(1)", xy=(1, np.cos(1)), xycoords="data", xytext=(+10, +30), textcoords="offset points",
                 # 标记箭头设置 标记符号 箭头弧度
                 arrowprops=dict(arrowstyle="->", connectionstyle="arc3,rad=.2"))

```

### 1显示中文

```python
plt.rcParams['font.sans-serif'] = ['SimHei']  # 用来正常显示中文标签
    plt.rcParams['axes.unicode_minus'] = False  # 用来正常显示负号
```



### 生成图表

```python
# 生成图表
    plt.show()
```



## 绘制余弦 正弦 图表

```python
'''
绘制余弦 正弦 
'''


def draw_cos_sin():
    # -np.pi, np.pi, 之间有 256点 包含最后一个点
    x = np.linspace(-np.pi, np.pi, 256, endpoint=True)

    # 余弦函数  # 正弦函数
    c, s = np.cos(x), np.sin(x)
    # 绘制表格
    plt.figaspect(1)
    # 绘制 自变量   因变量
    plt.plot(x, c, "r*")
    #  原线 自变量 颜色 线宽 线类型 标签 透明度
    plt.plot(x, s, color="blue", linewidth=1.0, linestyle="-", label="SIN", alpha=0.5)
    # 加入标题
    plt.title("cos&sin")
    # 加入网格线
    plt.grid()
    # 以上部分已经可以生成出图表了

    '''
    定位 边框 上下左右
    '''
    # 添加横轴纵轴编辑器 绘制基础点位置
    # 添加编辑器
    ax = plt.gca()
    # 定位上下左右图表

    # set_position 设置位置居中
    ax.spines['left'].set_position('center')
    # 左边的线 放入到数据域 0的这个位置
    # ax.spines['left'].set_position(("data", 0))
    ax.spines['bottom'].set_position('center')
    # set_color设置颜色无
    ax.spines['right'].set_color('none')
    ax.spines['top'].set_color('none')

    # 设置 x轴y轴参数 显示位置 底部显示x轴点 和左边显示y轴点
    ax.xaxis.set_ticks_position('bottom')
    ax.yaxis.set_ticks_position('left')

    # 设置横轴 横轴标示的位置  标示的内容
    plt.xticks([-np.pi, -np.pi / 2, 0, np.pi / 2, np.pi], [r'$-\pi$', r'$-\pi/2$', r'$0$', '$+\pi/2$', '$+\pi$'])
    # 设置纵轴 -1,1 5个点 并且显示
    plt.yticks(np.linspace(-1, 1, 5, endpoint=True))

    # 标记点 设置大小
    for label in ax.get_xticklabels() + ax.get_yticklabels():
        # 设置字体大小
        label.set_fontsize(16)
        # 设置label的小方块的格式  背景颜色 边缘  透明度
        label.set_bbox(dict(facecolor="yellow", edgecolor="none", alpha=0.2))

    # 图例图 就是左边有个标记样式的
    plt.legend(loc='upper left')

    # 控制显示范围 x轴最小 x轴最大 y轴最小 y轴最大
    # plt.axis([-1, 1, -0.5, 1])

    # 填充区域纵坐标     轴 范围 颜色 透明度
    plt.fill_between(x, 0, 0.5, color="green", alpha="0.53")
    # 填充区域纵坐标     轴 范围 颜色 透明度
    plt.fill_between(x, np.abs(x) < 0.5, c, c > 0.5, color="black", alpha="0.53")
    # 填充区域横坐标     轴 范围 颜色 透明度
    plt.fill_betweenx(x, 0, 0.5, color="red", alpha="0.53")

    # 标记点  在1,1的地方并且 在 0,np.cos(1)之间
    plt.plot([1, 1], [0, np.cos(1)], "y", linewidth=6, linestyle='-.', color="purple")

    # 标记点 添加注释    注释名称 点的位置   标记的偏移量 x+10 y+30   相对位置 箭头类型
    plt.annotate("cos(1)", xy=(1, np.cos(1)), xycoords="data", xytext=(+10, +30), textcoords="offset points",
                 # 标记箭头设置 标记符号 箭头弧度
                 arrowprops=dict(arrowstyle="->", connectionstyle="arc3,rad=.2"))

    # 生成图表
    plt.show()
```

## 绘制直线

```python

def draw_line():
    x = np.linspace(-5, 5, 256)
    y = 2 * x + 1
    # 绘制 '-r'表示上色
    '''
    颜色
b: blue 
g: green 
r: red 
c: cyan 
m: magenta 
y: yellow 
k: black 
w: white
'-.'+颜色 表示按颜色 -.-.-.-.
':'+颜色 表示 ......
'.'+颜色 表示 粗线的......
'--'+颜色 表示 - - - - - -
'-'+颜色 表示 ---------实线
'r*' 表示 ***********s
''
 
    '''
    plt.plot(x, y, '-r', label='y=2x+1')
    plt.plot(x, 2 * x - 1, '-.g', label='y=2x-1')
    plt.plot(x, 2 * x + 3, ':b', label='y=2x+3')
    plt.plot(x, 2 * x - 3, '--m', label='y=2x-3')
    plt.plot(x, 2 * x - 3, '.m', label='y=2x-3')
    plt.title("'Graph of y=2x+1'")
    plt.legend(loc='upper left')
    plt.grid()
    plt.show()

```

## 绘制散点图1

```python


def scatter_chart():
    
   # 调节图形大小，宽，高
    fig = plt.figure()

    # 散点数 根
    n = 128
    # x轴 随机数
    X = np.random.normal(0, 1, n)
    # y轴 随机数
    Y = np.random.normal(0, 1, n)
    # 上色
    T = np.arctan2(Y, X)
    # 显示范围 二选一
    '''
     # 设置 三行三列 第一格子
    ax = fig.add_subplot(3, 3, 1)
    # 绘制散点 x,y s=点的大小 c=颜色上色 ,透明度
    ax.scatter(X, Y, s=75, c=T, alpha=.5)
      
      或者
    plt.axes([0.025, 0.025, 0.95, 0.95])
     # 绘制散点 x,y s=点的大小 c=颜色上色 ,透明度
    plt.scatter(X, Y, s=75, c=T, alpha=.5)
    '''
    # 设置 三行三列 第一格子
    ax = fig.add_subplot(3, 3, 1)
    # 绘制散点 x,y s=点的大小 c=颜色上色 ,透明度
    ax.scatter(X, Y, s=75, c=T, alpha=.5)

    # x的范围
    plt.xlim(-1.5, 1.5), plt.xticks([])
    # y的范围
    plt.ylim(-1.5, 1.5), plt.yticks([])
    # 可以查看图形的x轴的最小最大坐标和y轴的最小最大坐标
    plt.axis()
    # 设置横轴坐标
    plt.xlabel("x")
    # 设置纵轴坐标
    plt.ylabel("y")
    # 标题
    plt.title("scatter")
    # 生成图表
    plt.show()
```

## 散点图2

```python
def scatter_chart_1():
    plt.rcParams['font.sans-serif'] = ['SimHei']  # 用来正常显示中文标签
    plt.rcParams['axes.unicode_minus'] = False  # 用来正常显示负号

    N = 100
    x = np.random.randn(N)
    y = np.random.randn(N)
    plt.scatter(x, y)

    plt.title("散点图示例01")  # 显示图表名称
    plt.xlabel("x轴")  # x轴名称
    plt.ylabel("y轴")  # y轴名称
    plt.text(+1.2, -1, "By:cat", fontsize=16, color="purple")

    plt.show()

```



## 绘制柱状图

```python


def bar_chart():
    # 绘制表格
    fig = plt.figure()

    fig.add_subplot(332)

    n = 10

    X = np.arange(n)

    Y1 = (1 - X / float(n)) * np.random.uniform(0.5, 1.0, n)

    Y2 = (1 - X / float(n)) * np.random.uniform(0.5, 1.0, n)

    plt.bar(X, +Y1, facecolor='#9999ff', edgecolor='white')

    plt.bar(X, -Y2, facecolor='#ff9999', edgecolor='white')

    # 添加注释
    for x, y in zip(X, Y1):
        ## 位置 格式 水平位置 垂直位置
        plt.text(x + 0.4, y + 0.05, '%.2f' % y, ha='center', va='bottom')
    for x, y in zip(X, Y2):
        plt.text(x + 0.4, -y - 0.05, '%.2f' % y, ha='center', va='top')

    # 可以查看图形的x轴的最小最大坐标和y轴的最小最大坐标
    plt.axis()
    # 设置横轴坐标
    plt.xlabel("x")
    # 设置纵轴坐标
    plt.ylabel("y")
    plt.show()


```

## 绘制饼图

```python

def pie_chart():
    plt.rcParams['font.sans-serif'] = ['SimHei']  # 用来正常显示中文标签

    # 调节图形大小，宽，高
    plt.figure(figsize=(6, 10))

    # 标签
    labels = ['娱乐', '育儿', '饮食', '房贷', '交通', '其它']
    # 数据
    sizes = [2, 5, 12, 70, 2, 9]
    # 每一块离开中心距离
    explode = (0, 0, 0, 0, 0, 0)
    # 自定义颜色列表
    colors = ['r', 'g', 'y', 'b']
    # 构造拼图   每一块离开中心距离  标签 控制饼图内百分比设置 控制饼图内百分比设置  显示阴影 起始绘制角度>90y轴正方向画起 自定义颜色
    patches, l_text, p_text = plt.pie(sizes, explode=explode, labels=labels, autopct='%1.1f%%', shadow=False, startangle=150, colors=colors)
    # 标题
    plt.title("饼图示例-8月份家庭支出")

    # 文本信息
    plt.text(1, -1.2, 'By:cat')

    # 改变文本的大小
    # 方法是把每一个text遍历。调用set_size方法设置它的属性
    for t in l_text:
        t.set_size(30)
    for t in p_text:
        t.set_size(20)
    # 添加图例
    plt.legend(loc="upper right", fontsize=10, bbox_to_anchor=(1.1, 1.05), borderaxespad=0.3)
    # loc =  'upper right' 位于右上角
    # bbox_to_anchor=[0.5, 0.5] # 外边距 上边 右边
    # ncol=2 分两列
    # borderaxespad = 0.3图例的内边距

    # 将饼图显示为正圆形
    plt.axis('equal')
    # 保存图表
    plt.savefig(r"E:\饼图02.png", dpi=200, bbox_inches='tight')
    # 显示图表
    plt.show()
```

## 折线图

```python
def polar_line():
    x = np.arange(2, 26, 2)  # x坐标
    y = [15, 13, 14, 5, 17, 20, 25, 26, 24, 22, 18, 15]
    # 设置图片大小
    plt.figure(figsize=(20, 8), dpi=80)  # figsize设置图片大小，dpi设置清晰度
    plt.plot(x, y, label='Y1')  # 绘制y1
    plt.plot(x, np.cos(y), lw=1, c='g', marker='o', label='Y2')  # 绘制y2
    # 设置x轴的刻度
    plt.xticks(x)
    # 设置y轴的刻度
    plt.yticks(range(min(y), max(y) + 1))  # 最后一位取不到，所以要加1

    # plt.xlim(0, 26)  # x轴坐标范围
    # plt.ylim(1, 100)  # y轴坐标范围
    plt.xlabel('X-Name')  # x轴标注
    plt.ylabel('Y-Name')  # y轴标注
    plt.legend()  # 图例
    # 保存
    # plt.savefig("./t1.png")
    plt.show()
```

## 圆形图

```python
'''
圆形图
'''
def roundness_chart():
    x = np.arange(2, 26, 2)  # x坐标
    y = [15, 13, 14, 5, 17, 20, 25, 26, 24, 22, 18, 15]
    # 设置图片大小
    plt.figure(figsize=(20, 8), dpi=80)  # figsize设置图片大小，dpi设置清晰度
    # 更改图形为圆形
    plt.polar(x, y, label='Y1')  # 绘制y1
    plt.polar(x, np.cos(y), lw=1, c='g', marker='o', label='Y2')  # 绘制y2
    # 设置x轴的刻度
    plt.xticks(x)
    # 设置y轴的刻度
    plt.yticks(range(min(y), max(y) + 1))  # 最后一位取不到，所以要加1

    # plt.xlim(0, 26)  # x轴坐标范围
    # plt.ylim(1, 100)  # y轴坐标范围
    plt.xlabel('X-Name')  # x轴标注
    plt.ylabel('Y-Name')  # y轴标注
    plt.legend()  # 图例
    # 保存
    # plt.savefig("./t1.png")
    plt.show()
```

## 热力图

需要引入新模块

```python
from matplotlib import cm
```

```python

def heatmap_chart():
    from matplotlib import cm
    # 设置图片大小
    plt.figure(figsize=(20, 8), dpi=80)  # figsize设置图片大小，dpi设置清晰度
    # 绘制的数据 3*5的随机数
    data = np.random.rand(3, 5)
    # 颜色
    cmap = cm.Blues
    # 构建热力图   相邻的相同的颜色连成片   自动缩放 颜色最大值 最小值设置
    map = plt.imshow(data, interpolation='nearest', cmap=cmap, aspect='auto', vmax=1, vmin=0)
    plt.show()
```



## 3D图

需要引入新模块

```python
from mpl_toolkits.mplot3d import Axes3D
```

```python
'''
3D图
'''
def three_chart():
    from mpl_toolkits.mplot3d import Axes3D
    # 绘制画布
    fig = plt.figure()
    # 创建3D
    axes3d = Axes3D(fig)
    x = np.arange(2, 26, 2)  # x坐标
    y = [15, 13, 14, 5, 17, 20, 25, 26, 24, 22, 18, 15]
    # 绘制散点
    axes3d.scatter3D(x, y, np.log(x + y))
    plt.show()
```

## 热图

```python
''''
热图
'''


def hot_chart():
    # 绘制画布
    fig = plt.figure()
    # 计算x,y坐标对应的高度值
    def f(x, y):
        return (1 - x / 2 + x ** 3 + y ** 5) * np.exp(-x ** 2 - y ** 2)
    n = 256
    # 生成x,y的数据
    x = np.linspace(-3, 3, n)
    y = np.linspace(-3, 3, n)
    # 把x,y数据生成mesh网格状的数据，因为等高线的显示是在网格的基础上添加上高度值
    X, Y = np.meshgrid(x, y)
    # 绘制热图 填充等高线
    plt.contourf(X, Y,f(X,Y),  8, alpha=.75, cmap=plt.cm.hot)
    plt.show()
```

## 其他图例

```python

```

