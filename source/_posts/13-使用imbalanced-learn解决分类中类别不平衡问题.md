---
title: 13.使用imbalanced-learn解决分类中类别不平衡问题
date: 2026-03-02 14:29:39
tags: [机器学习]
---

# 1.使用imbalanced-learn解决分类中类别不平衡问题

在分类算法中会存在 一部分类别的数据大于1/4 导致了两部分类别数据的比例 为1:4
`imbalanced-learn` 是用来处理这部分数据的

## 2.导入依赖

    pip install imbalanced-learn

    conda install imbalanced-learn

<!--more-->

## 3.案例 make\_classification构建模拟不平衡的数据集

```
# 使用lmblearn包处理数据不平衡问题
from sklearn.datasets import make_classification
from sklearn.model_selection import train_test_split
import matplotlib.pyplot as plt

# 使用make_classification生成样本数据
X,y=make_classification(n_samples=5000,
                        n_features=2, #特征数量
                        n_informative=2, #多信息特征个数
                        n_redundant=0,#冗余信息
                        n_repeated=0,#重复信息
                        n_classes=3,#分类类别
                        n_clusters_per_class=1,#某一个类别是由几个cluster构成的
                        weights=[0.01,0.05,0.94], #列表类型,权重比
                        random_state=0)

# 查看各个标签的样本
from collections import Counter
print(Counter(y))

# 数据可视化
plt.scatter(X[:,0],X[:,1],c=y)
plt.show()


# 划分训练集和测试集
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.3, random_state=42)


```

## 4解决类别不平衡数据方法

#### 4.1主要两种解决方式

*   过采样方式  (一般用这个)
    \*\* 增加数据较少的那一类样本的数据,是的正负样本比例均衡

*   欠采样方式
    \*\* 减少数据较多的那一类样本的数据,是的正负样本比例均衡

#### 4.2 过采样方法

对训练集里的少数类进行"过采样"(oversampling),
`即增加一些少量类样本使得 正丶反例数目接近,然后再进行学习`

###### 4.2.1  随机过采样方法  RandomOverSampler

也就是`在少数类样本中随机抽取一些数据,通过复制的方式生成数据集E`,然后扩大原始数据类集合

`from imblearn.over_sampling import RandomOverSampler`

```
# 使用lmblearn包处理数据不平衡问题
from sklearn.datasets import make_classification
from sklearn.model_selection import train_test_split
import matplotlib.pyplot as plt
from collections import Counter
## imblearn随机过采样方法
from imblearn.over_sampling import RandomOverSampler

# 使用make_classification生成样本数据
X,y=make_classification(n_samples=5000,
                        n_features=2, #特征数量
                        n_informative=2, #多信息特征个数
                        n_redundant=0,#冗余信息
                        n_repeated=0,#重复信息
                        n_classes=3,#分类类别
                        n_clusters_per_class=1,#某一个类别是由几个cluster构成的
                        weights=[0.01,0.05,0.94], #列表类型,权重比
                        random_state=0)

# 查看原始数据-各个标签的样本
print(Counter(y))

# 使用随机过采样
ros=RandomOverSampler(random_state=0)
X_resampled,y_resampled=ros.fit_resample(X,y)

# 查看随机过采样后-各个标签的样本
print(Counter(y_resampled))

# 原始数据-数据可视化
plt.scatter(X[:,0],X[:,1],c=y)
plt.show()

# 使用随机过采样后-数据可视化
plt.scatter(X_resampled[:,0],X_resampled[:,1],c=y_resampled)
plt.show()

```

输出结果:
原始数据-各个标签的样本: `Counter({2: 4674, 1: 262, 0: 64})`
随机过采样后-各个标签的样本: `Counter({2: 4674, 1: 4674, 0: 4674})`

*   <font color="red">缺点:</font>

*   对于随机过采样,由于需要对少数类样本进行复制来扩大数据集,造成`模型训练复杂度加大`
    \*另一方面也容易`造成模型的过拟合问题`, 因为随机过采样是简单的对初始样本数据进行复制采样,这就使得学习器学到的规则过于具体化,不利于学习器的泛化性能,造成过拟合问题

*   <font color="red">处理方案: SMOTE算法</font>

###### 4.2.2  随机过采样-SMOTE算法    (一般用这个)

保证数据集均衡 ,不造成模型过拟合问题

    from imblearn.over_sampling import SMOTE
    from imblearn.pipeline import Pipeline

```
# 使用lmblearn包的SMOTE处理数据不平衡问题-保证数据集均衡
from imblearn.over_sampling import SMOTE
from imblearn.pipeline import Pipeline


from sklearn.datasets import make_classification
from sklearn.model_selection import train_test_split
import matplotlib.pyplot as plt
from collections import Counter


# 使用make_classification生成样本数据
X,y=make_classification(n_samples=5000,
                        n_features=2, #特征数量
                        n_informative=2, #多信息特征个数
                        n_redundant=0,#冗余信息
                        n_repeated=0,#重复信息
                        n_classes=3,#分类类别
                        n_clusters_per_class=1,#某一个类别是由几个cluster构成的
                        weights=[0.01,0.05,0.94], #列表类型,权重比
                        random_state=0)

# 初始化 SMOTE
smote = SMOTE(random_state=42)
X_resampled,y_resampled=smote.fit_resample(X,y)

# 查看原始数据-各个标签的样本
print("原始数据-各个标签的样本:",Counter(y))

# 查看随机过采样后-各个标签的样本
print("SMOTE保证数据集均衡后-各个标签的样本:",Counter(y_resampled))

# 原始数据-数据可视化
plt.scatter(X[:,0],X[:,1],c=y)
plt.show()

# 使用随机过采样后-数据可视化
plt.scatter(X_resampled[:,0],X_resampled[:,1],c=y_resampled)
plt.show()

```

#### 4.3 欠采样方法

直接对训练集中多数类样本进行`欠采样`(underSampling),
即`去除一些多数类中的样本使得正丶反例数目接近,然后再进行学习`

###### 4.3.1  随机欠采样方法  RandomUnderSampler

    from imblearn.under_sampling import RandomUnderSampler

```
# 使用lmblearn包处理数据不平衡问题
from sklearn.datasets import make_classification
from sklearn.model_selection import train_test_split
import matplotlib.pyplot as plt
from collections import Counter
## imblearn随机过采样方法
from imblearn.under_sampling import RandomUnderSampler

# 使用make_classification生成样本数据
X,y=make_classification(n_samples=5000,
                        n_features=2, #特征数量
                        n_informative=2, #多信息特征个数
                        n_redundant=0,#冗余信息
                        n_repeated=0,#重复信息
                        n_classes=3,#分类类别
                        n_clusters_per_class=1,#某一个类别是由几个cluster构成的
                        weights=[0.01,0.05,0.94], #列表类型,权重比
                        random_state=0)

# 查看原始数据-各个标签的样本
print("原始数据-各个标签的样本:",Counter(y))

# 使用随机欠采样
ros=RandomUnderSampler(random_state=0)
X_resampled,y_resampled=ros.fit_resample(X,y)

# 查看随机过采样后-各个标签的样本
print("随机过采样后-各个标签的样本:",Counter(y_resampled))

# 原始数据-数据可视化
plt.scatter(X[:,0],X[:,1],c=y)
plt.show()

# 使用随机欠采样后-数据可视化
plt.scatter(X_resampled[:,0],X_resampled[:,1],c=y_resampled)
plt.show()


# 划分训练集和测试集
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.3, random_state=42)


```

*   <font color="red">缺点:</font>
*   随机欠拟合方法通过改变多数类样本的数量,使其样本分布较为均衡
*   由于采样的样本集合要少于原来的样本集合,因此会导致一些信息缺失,即`多数样本删除有可能会导致分类器丢失有关多数类的重要信息`

