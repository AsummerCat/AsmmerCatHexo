---
title: python-机器学习分类算法knn模型实现
date: 2020-01-28 20:39:30
tags: [python,机器学习,数学]
---

# knn算法

又称 k-近邻算法 是`一种基本分类和与回归方法`

![knn算法](/img/2020-01-28/3.png)

## 常用分类算法

![常用分类算法](/img/2020-01-28/1.png)

<!--more-->

## knn算法思想

![knn算法思想](/img/2020-01-28/2.png)



## 普通demo

```python
# -*- coding: utf-8 -*-
'''
knn模型预测天气冷暖
'''
import operator

import numpy as np
from matplotlib import pyplot as plt
import math


# 创建数据集 返回特征向量 和类标签
def create_dataset():
    # 特征向量 数据集合 一般情况下 是经过数据预处理后生成的特征向量
    # 冰淇淋数量 喝水数量 户外活动时长
    dataset = np.array([[8, 4, 2], [7, 1, 1], [1, 4, 4], [3, 0, 5]])
    # 类标签
    # 冷热程度
    labels = ['非常热', '非常热', '一般热', '一般热']
    return dataset, labels


# 可视化数据处理
def analyze_data_plot(x, y):
    # 方式一
    ## 画布
    fig = plt.figure()
    ## 将画布划分为1行1列1块
    ax = fig.add_subplot(111)
    ax.scatter(x, y)
    ## 散点图标题及其横纵标的名称
    plt.title('scatter chart')
    plt.xlabel("x")
    plt.ylabel("y")
    plt.show()

    # 方式二
    # plt.scatter(x, y)
    # plt.show()


# 构造knn分类器
'''
 新样本   new_value
样本库数据 dataset labels
k值  k_value
'''


def knn_classifier(new_value, dataset, labels, k):
    # 1.获取新的样本数据  new_value
    # 2.获取样本库的数据  dataset labels
    # 3.选择k值  k_value

    # 4.计算样本数据和样本库数据之间的距离 欧式距离计算
    sqrt_dist = optimize_compute_euclidean_distance1(new_value, dataset)
    # 5.根据距离排序 按照列向量排序   升序  axis=0 列   获取出索引
    sort_dist_index_s = sqrt_dist.argsort(axis=0)
    # 6.针对k个点,统计各个类别的数量
    class_count = {}  # 统计各个类别分别是多少
    for i in range(k):
        # 根据距离排序索引值 获取类标签
        vote_label = labels[sort_dist_index_s[i]]  # 获取排序距离
        # print(sort_dist_index_s[i], vote_label)
        # 统计标签各个的个数 键值对
        class_count[vote_label] = class_count.get(vote_label, 0) + 1
    # 7.投票 少数服从多数 输出类别 (k的类别中哪个多,选哪个)
    ## key=operator.itemgetter(1) 根据值进行排列  itemgetter(0)根据列进行排列 默认降序
    sort__class_count = sorted(class_count.items(), key=operator.itemgetter(1), reverse=True)
    ## 获取获取排序后的最多的那个key 也就是label
    predict_value = sort__class_count[0][0]
    # print("新值:", new_value, ",KNN投票预测结果是:", predict_value)
    return predict_value


# 欧式距离计算: d=(x1-x2)的平方+(y1-y2)的平方 然后进行开方
def compute_euclidean_distance(x1, y1, x2, y2):
    d = math.sqrt(math.pow((x1 - x2), 2) + math.pow((y1 - y2), 2))
    return d


# 优化1 欧式距离计算 如果有多个xy 那就+在后面  可支持三维以上计算   [x,y,z] [x,y,z]
def optimize_compute_euclidean_distance(instance1, instance2, length):
    d = 0
    for x in range(1, length):
        d + pow(instance1[x], instance2[x], 2)
    return math.sqrt(d)


# 优化2 欧式距离计算  优化上千维度 (推荐)
def optimize_compute_euclidean_distance1(new_value, dataset):
    # 获取数据向量dataset的长度
    row_size, col_size = dataset.shape
    # 将新的数据 各特征向量 进行比较 矩阵计算
    diff_mat = np.tile(new_value, (row_size, 1)) - dataset
    # 对差值平方
    sqrt_mat = diff_mat ** 2
    # 差值开方 求和
    sqrt_dist = sqrt_mat.sum(axis=1) ** 0.5
    return sqrt_dist


def test():
    # 1.创建数据集 和类标签
    dataset, labels = create_dataset()
    print('数据集:', dataset, '\n', labels)
    # 2.可视化数据分析

    # 测试第一列 第二列数据
    analyze_data_plot(dataset[:, 0], dataset[:, 1])

    # 3. 获取新数据
    new_value = np.array([2, 4, 0])

    # 欧式距离计算 测试
    # compute_euclidean_distance(1, 2, 7, 8)
    # 欧式距离计算优化1 测试
    # optimize_compute_euclidean_distance([1, 2, 1], [7, 8, 6], 3)
    # 欧式距离计算优化2 测试 推荐
    # optimize_compute_euclidean_distance1([1, 2, 1], dataset)
    # 4.构造knn分类器
    ## k值
    k_value = 3
    ## 预测比对结果
    predict_value = knn_classifier(new_value, dataset, labels, k_value)
    print("新值:", new_value, ",KNN投票预测结果是:", predict_value)

    # 批量预测
    vars_value = [[2, 2, 2], [2, 3, 4], [6, 6, 6], [9, 9, 1]]
    for i in vars_value:
        predict_value = knn_classifier(i, dataset, labels, k_value)
        print("新值:", i, ",KNN投票预测结果是:", predict_value)


if __name__ == '__main__':
    test()

```

## scikit-learn实现knn模型demo

```python
# -*- coding: utf-8 -*-
'''
近邻算法
'''
from sklearn import neighbors
import numpy as np

'''
knn预测算法
'''


def knn_sk_learn_predict():
    # 加载  算法 knn分类算法
    knn = neighbors.KNeighborsClassifier()
    # 获取数据集 类标签
    dataset, labels = create_dataset()
    # 训练模型
    knn.fit(dataset, labels)
    # 分类预测
    predictRes = knn.predict(np.array([[2, 4, 0]]))
    print("输出预测结果", predictRes)


# 创建数据集 返回特征向量 和类标签
def create_dataset():
    # 特征向量 数据集合 一般情况下 是经过数据预处理后生成的特征向量
    # 冰淇淋数量 喝水数量 户外活动时长
    dataset = np.array([[8, 4, 2], [7, 1, 1], [1, 4, 4], [3, 0, 5], [1, 5, 5]])
    # 类标签
    # 冷热程度
    labels = ['非常热', '非常热', '一般热', '一般热', '一般热']
    return dataset, labels


if __name__ == '__main__':
    knn_sk_learn_predict()

```

