---
title: python-机器学习scikit-learn库
date: 2020-01-13 21:45:04
tags: [python,机器学习]
---

# scikit-learn库(常用)

关键字 : 数据挖掘 机器学习   高级数据分析常用工具包

## 官网

http://scikit-learn.org

## [demo地址](https://github.com/AsummerCat/scikit-learn-demo)

## 机器学习的概念

![概念](/img/2020-01-13/4.png)

不打标记: 无监督学习 例如: 根据属性差不多的放在一个类中

打标记:监督学习 根据标记进行分类

<!-more-->

## 数据挖掘和机器学习

大致分为三步骤

```python
数据预处理   -> 降维处理   向量化
数据建模
数据验证
```



## 决策树

![决策树](/img/2020-01-13/5.png)

每个内部节点 表示属性上的测试

每个分支   表示一个测试输出

每个叶节点 表示一种类别

# 测试

## 数据集

![数据集](/img/2020-01-13/7.png)

## 测试

```python
# 数据预处理
    '''
    这边鸢尾已经在scikit库中 直接调用
    '''
    # 直接引入鸢尾iris数据集
    from sklearn.datasets import load_iris
    # data:属性 target:标注
    iris = load_iris()

    # 获取属性
    # print(len(iris["data"]))
    # 因为这边数据是比较规则的 不用预处理

    # 预处理
    ## 需要导入模块
    from sklearn.model_selection import train_test_split
    ## 将数据分为测试数据 和验证数据   test_size表示测试数据占比20% random_state=1 随机选择30个数据
    train_data, test_data, tarin_target, test_target = train_test_split(iris.data, iris.target, test_size=0.2,
                                                                        random_state=1)

    # 建立模型
    ## 需要导入模块
    from sklearn import tree
    ## 建立决策树 分类器 (回归器:DecisionTreeRegressor)    criterion:标准选择 信息熵
    clf = tree.DecisionTreeClassifier(criterion="entropy")
    # 建立模型 ->训练集训练
    clf.fit(train_data, tarin_target)
    # 进行预测
    y_pred = clf.predict(test_data)

    # 验证
    ## 需要导入模块
    from sklearn import metrics
    ## 两种方式

    ### 准确率
    #### y_true 验证的数据  y_pred:预测值
    ##### 输出结果就是 准确率
    print(metrics.accuracy_score(y_true=test_target, y_pred=y_pred))

    ### 混淆矩阵->误差矩阵
    print(metrics.confusion_matrix(y_true=test_target, y_pred=y_pred))

    ## 决策树输出结构文件
    with open("./data/tree.dot", "w") as fw:
        tree.export_graphviz(clf, out_file=fw)
```



