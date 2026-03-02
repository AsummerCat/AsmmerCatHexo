---
title: 21.特征工程-支持向量机SVM算法
date: 2026-03-02 14:35:25
tags: [机器学习]
---

# 支持向量机(SVM)算法

跟逻辑回归一样是 一个判别式模型
是一种强大的`监督学习算法`，广泛应用于分类和回归任务

SVM 的核心思想是通过找到`一个最优的超平面来分割不同类别的数据点`，使得不同类别的数据点在超平面两侧的距离最大化。这样可以提高模型的泛化能力，减少过拟合的风险。

是机器学习最受欢迎的模型之一,特别适用于中小型复杂数据集的分类

<!--more-->

简单描述就是: 超平面分为两类,并且间隔最大

*   `损失函数`：帮助我们评估模型的好坏，特别是数据点离超平面的距离和是否被正确分类。
*   `核方法`：帮助我们在高维空间中找到一个超平面，即使在原始空间中数据点是线性不可分的。

## 1.SVM.SVC使用 (分类)

超参数 `C`: 用来控制`软平衡`,
可以当做`C是罚钱的力度`,
`C值`越小,间隔越宽(宽的话,存在间隔内的违例数据就越多), (相对错误率高一点)
`C值`越大,间隔越窄(窄的话,存在间隔内的违例数据就越少),(错误率比较低)

C值越小,泛化效果越好

*   API

<!---->

    from sklearn import svm

    x=[[0,0],[1,1]]
    y=[0,1]

    clt=svm.SVC(C=1.0,kernel='linear')

    # 训练
    clt.fit(x,y) 

    #预测
    clt.predict([[2,2]])

*   API参数定义

```
** C: 正则化参数。
C 控制误分类的惩罚力度 ,平衡模型的复杂度和训练误差。
默认值为 1.0 
** kernel: 核方法名称 比如kernel='linear' 默认值为`rbf`
   有 'linear'：线性核
      'poly'：多项式核
      'rbf'：高斯核（径向基函数核）
      'sigmoid'：Sigmoid核
      'precomputed'：预计算核矩阵

** degree: 多项式核函数的次数 仅在 kernel='poly' 时有效，控制多项式核的复杂度

** gamma: 高斯核（RBF核）和多项式核的系数

** cache_size:指定内核缓存的大小（以 MB 为单位）
大的缓存可以加快训练速度，尤其是在处理大规模数据集时


```

## 2. 损失函数

损失函数就像是一个评分系统，它会告诉你模型做得好不好。在SVM中，我们希望找到一个超平面，使得不同类别的数据点在这个超平面两侧的距离尽可能大。但是，实际数据往往不是完美的，可能会有一些点离超平面太近，甚至出现在了错误的一侧。这时候，损失函数就会给出一个“惩罚”，告诉我们这些点的存在对模型的影响有多大。

`Hinge Loss`
SVM 使用的损失函数叫做` Hinge Loss`。
`Hinge Loss` 的作用是：

*   如果一个数据点被正确分类且距离超平面足够远，那么它的损失值为0。
*   如果一个数据点被正确分类但距离超平面太近，或者被错误分类，
*   那么它的损失值会是一个正数，表示这个点对模型的负面影响。

<!---->

    from sklearn.metrics  import hinge_loss

    hinge_loss()

*   测试案例

<!---->

    from sklearn.svm import LinearSVC
    from sklearn.metrics import hinge_loss
    from sklearn.model_selection import train_test_split
    from sklearn.datasets import make_classification
    import numpy as np

    # 生成模拟数据
    X, y = make_classification(n_samples=100, n_features=20, n_classes=2, random_state=42)

    # 将标签转换为 +1 和 -1
    y = np.where(y == 0, -1, 1)

    # 划分训练集和测试集
    X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

    # 实例化线性SVM分类器
    clf = LinearSVC(loss='hinge', random_state=42)

    # 训练模型
    clf.fit(X_train, y_train)

    # 获取模型的决策函数值
    decision_values = clf.decision_function(X_test)

    # 计算铰链损失
    loss = hinge_loss(y_test, decision_values)

    print(f"Hinge Loss: {loss:.4f}")

## 3. 核方法

核方法的核心思想是，我们不需要显式地把数据点从低维空间映射到高维空间，而是通过核函数直接计算两个数据点在高维空间中的相似度。这样做的好处是，计算量大大减少，同时还能处理非常复杂的非线性关系。

(kernel)核方法分为

*   线性核: 线性核是最简单的核函数，适用于线性可分的数据`SVC(kernel='linear')`
*   使用多项式核: 多项式核可以捕捉更复杂的非线性关系`SVC(kernel='poly', degree=3)`
*   使用高斯核（RBF核）:高斯核是最常用的核函数，适用于非常复杂的非线性关系。 `SVC(kernel='rbf', gamma='scale')`
*   使用Sigmoid核: Sigmoid核类似于神经网络中的激活函数 `SVC(kernel='sigmoid')`

## 4.SVM.SVR (回归)

跟SVM做分类是有差异的
`SVM回归`是尽可能多的实例位于预测线上的,同时限制间隔违例(也就是不在预测线距上的实例)

支持向量回归（Support Vector Regression, SVR）是支持向量机（SVM）在回归任务中的应用。SVR 的目标是找到一个函数，使得该函数与实际值之间的偏差在一定范围内尽可能小。SVR 通过引入一个不敏感损失函数（ε-insensitive loss function）来实现这一点。

    import numpy as np
    import matplotlib.pyplot as plt
    from sklearn.svm import SVR
    from sklearn.model_selection import train_test_split
    from sklearn.metrics import mean_squared_error
    # 实例化线性核的SVR模型
    svr_linear = SVR(kernel='linear', C=1.0, epsilon=0.1)

    # 实例化多项式核的SVR模型
    svr_poly = SVR(kernel='poly', C=1.0, degree=3, epsilon=0.1)

    # 实例化RBF核的SVR模型
    svr_rbf = SVR(kernel='rbf', C=1.0, gamma='scale', epsilon=0.1)

## 5.SVM算法API综述

*   SVM算法具有良好的鲁棒性,对未知数据拥有很强的泛化能力,`特别是在数据较少的情况下`,相较其他传统机器学习算法具有更优的性能
*   使用SVM作为模型的时候,通常采用以下流程

```
1.对样本数据进行归一化
2.应用核函数对样本进行映射
(最常采用核函数是RBF和Linear,在样本线性可分时,linear比RBF效果要好)
3.用cross-vaidation(交叉验证)和grip-search(网格搜索)对超参数进行优选
4.用最优参数进行训练模型
5.测试

```

*   sklearn中支持主要方法
    `向量分类`:SVC ,NuSVC , LinearSVC

NuSVC的nu参数=SVC的C参数
LinearSVC是实现线性核函数的支持向量分类,没有kernel参数

`向量回归`:SVR,NuSVR,LinearSVR

\*\* SVC:

    sklearn.svm.SVC(C=1.0,kernel='rbf',degree=3,coef0=0.0,random_state=None)
    c值越小惩罚力度越小
    c值越大惩罚力度越大 
    C=惩罚力度越大,准确率很高,但泛化能力弱,容易导致过拟合
    C=惩罚力度越小,容错能力增大,泛化能力较强 ,但也可能出现欠拟合

\*\* NuSVC:

    sklearn.svm.SVC(nu=1.0)
    nu: 训练误差部分的上限和支持向量部分的下限,取值在(0,1)之间,默认是0.5

\*\* LinearSVC:

    sklearn.svm.LinearSVC(penalty='l2',loss='squared_hinge',dual=True,C=1.0)
    参数: 
    penalty: 正则化参数 ,可选 l1 l2 ,仅LinearSVC有
    loss: 损失函数,有hinge和squared_hinge可选, 前者是 L1损失,后者是L2损失 ,默认为squared_hinge
          其中hinge是SVM的标准损失,squared_hinge是hinge的平方
          
    dual: 是否转换为对偶问题求解,默认是True

    C: 惩罚系数

