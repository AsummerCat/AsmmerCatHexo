---
title: 9.scikit-learn工具的使用
date: 2026-03-02 14:26:21
tags: [机器学习]
---

# scikit-learn工具的使用

py的机器学习工具
包含许多机器学习算法的实现

### 1.安装
```
    pip install scikit-learn==0.19.1

    conda install scikit-learn
```
执行命令查看是否安装成功
```
    import sklearn
```
<!--more-->

## 2.scikit-learn实现K近邻算法

*   API
    sklearn.neighbors.KNeighborsClassifier(n\_neighbors=5,algorithm='auto')

    参数: n\_neighbors  int默认等于5 , 邻居数=5
    algorithm :
    默认为auto 让算法自己决定使用的搜索方法
    brute:蛮力搜索,也就是线性扫描,当训练集很大的时候,计算非常耗时
    kd\_tree:构建kd树存储数据以便对其进行快速检索的树形数据结构
    ball\_tree: 克服kd树高纬失效的

    from sklearn.neighbors import KNeighborsClassifier

    # 构造数据集

    x= \[\[1],\[10],\[20],\[40],\[50]]
    y= \[0,0,1,1,2]

    # 机器学习模型训练

    # 实例化API->实例化一个估计器对象

    estimator =KNeighborsClassifier(n\_neighbors=1)

    # 使用fit方法进行训练

    ## x:特征值 y:目标值

    estimator.fit(x,y)

    ## 预测

    ret=estimator.predict(\[\[4]])

    print(ret)

    ## 返回的结果\[0] 接近\[1]

## 3. 使用scikit-learn 来获取数据集

```
sklean.datasets
 * 加载获取流行数据集
 
 *datasets.load_()
    获取小规模数据集
    
 *datasets.fetch_*(data_home=None)
  获取大规模数据集,需要从网络上下载, data_home表示数据集下载的目录
  `默认为 ~/scikit_learn_data/`

```

#### 3.1 加载测试数据集
```
    from  sklearn.datasets import load_iris
    import pandas as pd

    # sklearn小数据集
    # sklearn.datasets.load_iris() #加载并返回iris数据集
    iris=load_iris();

    # sklearn大数据集
    # sklearn.datasets.fetch_20newsgroups(home=None,subset='train')
    ## subset: 'train'或'test' 'all' 可选 ,选择要加载的数据集
    ## 训练集的'训练' ,训练集的'测试' , 两者的全部

    print(iris)
    # 数据集返回值介绍
    print(iris.data)
    ## data :特征数据数组

    print(iris["target"])
    ## target: 标签数组

    print(iris.DESCR)
    ## DESCR: 数据描述

    print(iris.feature_names)
    ## feature_names: 特征名,新闻数据,手写数字,回归数据集没有

    print(iris.target_names)
    ## target_names:标签名

    ## 加载数据集转换为DataFrame
    iris_d=pd.DataFrame(iris.data,columns=['sepal length in cm','sepal width in cm','petal length in cm','petal width in cm'])
    ## 并且加载目标值到二维数组里
    iris_d['target']=iris.target
    ## 预览前几行
    iris_d.head()
```
## 4.训练集划分 训练数据和测试数据

*   训练数据: 用于训练, 构建模型
*   测试数据: 在模型校验时使用,用于评估模型是否有效

划分比例:

*   训练集: 70% 80% 75%
*   测试集: 30% 20% 25%

#### 4.1 训练集划分API

```
sklearn.model_selection.train_test_split(arrays,*options)
参数: 
  x数据集的特征值
  
  y数据集的标签值
  
  test_size测试集的大小,一般为float  
  0.2的话表示测试集占比20%,训练集占比80%
  
  train_size训练集的大小,一般为float   0.2的话表示训练集占比20%,测试集占比80%
  
  random_state 随机数终止,不同的种子会造成不同的随机采样结果.相同的种子采样结果相同
  
return 
 x_train,x_test,y_train,y_test
  
```

*   案例:

```
# 数据集划分
from  sklearn.datasets import load_iris
# 数据集划分
from sklearn.model_selection import train_test_split
import pandas as pd

## 加载数据集
iris=load_iris()

## 对数据集进行划分
#### 训练集的特征数据x_train,测试集的特征数据x_test,训练集的目标标签(值)y_train,,测试集的目标标签(值)y_test , test_size表示测试集占比 0.2/训练集占比0.8 
x_train,x_test,y_train,y_test=train_test_split(iris.data,iris.target,test_size=0.2,random_state=22)
print(y_train.shape)
print(y_test.shape)

```

#### 4.2 训练集划分 注意

*   常见做法:大于2/3或者3/4的样本用于训练 剩余样本用于测试

需要进行分层采样

需要避免划分后的数据分布不一致性
比如:300个True 和150 False  被切分后 数据的比例不一致

*   这里可用`留出法`来实现

```
```

## 5. 特征工程-特征预处理

简单描述就是:
通过一些转换函数将特征数据转换城更加适合算法模型的特征数据过程

数字型数据的无量纲化

*   归一化
*   标准化

| 特征1 | 特征2 | 特征3 | 特征4 |
| --- | --- | --- | --- |
| 90  | 2   | 10  | 40  |
| 60  | 4   | 15  | 45  |
| 75  | 3   | 13  | 46  |

转换为->

| 特征1 | 特征2 | 特征3 | 特征4  |
| --- | --- | --- | ---- |
| 1.  | 0.  | 0.  | 0.   |
| 0.  | 1.  | 1.  | 0.83 |
| 0.5 | 0.5 | 0.6 | 1.   |

ps: 为什么要进行归一化/标准化
特征的单位或者大小相差较大,或者某特征的方差相比其他特征要大出几个数量级,容易影响(支配)目标结果,是的一些算法无法学习到其他特征

##### 5.1 特征预处理API

`sklearn.preprocessing`

###### 5.1.1 归一化处理

*   归一化处理
    定义: 通过对原始数据进行变换把数据映射到(默认为\[0,1])之间
    过程: 作用于每一列,max为一列的最大值,min为一列的最小值.
    场景: 因为最大值和最小值非常容易收到异常点的影响,所以这种方法稳定性较差,只适合传统精确小数据场景

API:
sklearn.preprocessing.MinMaxScaler(feature=(0,1)...)

*   MinMaxScaler.fit\_transform(X)

*   X\:numpy array格式的数据

*   返回值: 转换后形状相同的array

*   数据计算

```
# 数据特征处理-归一化

import pandas as pd
from sklearn.preprocessing import MinMaxScaler

data=pd.read_csv("./dating.csv")

print(data)

## 1.实例化一个转换器
transfer=MinMaxScaler(feature_range=(2,3))

## 2.调用
tres=ransfer.fit_transform(data[['A字段','B字段','C字段']])
print("最小值最大值归一化处理的结果:\n",tres)

```

###### 5.1.2 标准化处理 (常用)

*   标准化处理

定义: 通过原始数据进行变换,把数据变换为均值为0,标准差为1范围内

过程: 作用于每一列,mean为平均值 ,@为标准差 ,具体计算为 (列值-mean)/标准差

场景:
对于标准化而言:如果出现异常点,由于具有一定数据量,少量的异常点对平均值的影响并不大,从而方差改变较小
适用于: 在已有样本足够多的情况下比较稳定,适合现代嘈杂大数据场景

API:
sklearn.preprocessing.StandardScaler()

*   处理之后每列来说所有数据都聚集在均值0附近标准差 差为1

*   StandardScaler.fit\_transform(X)

*   X\:numpy array格式的数据

*   返回值: 转换后形状相同的array

*   数据计算

```
# 数据特征处理-标准化

import pandas as pd
from sklearn.preprocessing import StandardScaler

data=pd.read_csv("./dating.csv")

print(data)

## 1.实例化一个转换器 
transfer=StandardScaler()

## 2.调用fit_transform
res=transfer.fit_transform(data[['特征A','特征B','特征C']])
print("标准化处理的数据:\n",res.data)
print("每一列的方差为:\n",transfer.var_)
print("每一列的平均值:\n",transfer.mean_)

```

## 6.完整案例

```
# 数据集划分
from  sklearn.datasets import load_iris
# 数据集划分
from sklearn.model_selection import train_test_split
# 数据特征处理-标准化
from sklearn.preprocessing import StandardScaler
# KNN算法
from sklearn.neighbors import KNeighborsClassifier

import pandas as pd

## 加载数据集
iris=load_iris()

## 对数据集进行划分
#### 训练集的特征数据x_train,测试集的特征数据x_test,训练集的目标标签(值)y_train,,测试集的目标标签(值)x_test , test_size表示测试集占比 0.2/训练集占比0.8 
x_train,x_test,y_train,y_test=train_test_split(iris.data,iris.target,test_size=0.2,random_state=22)

## 模型标准化
ransfer=StandardScaler()
x_train=ransfer.fit_transform(x_train)
x_test=ransfer.transform(x_test)

## 模型预测
estimater=KNeighborsClassifier(n_neighbors=5,algorithm="ball_tree")
estimater.fit(x_train,y_train)

## 模型评估
#### 方法1:对比真实值和预测值
y_predict=estimater.predict(x_test);
print("预测结果为:\n",y_predict)
print("对比真实值和预测值:\n",y_predict==y_test)

#### 方法2:直接计算准确率 传入测试集的x,y(为一组包含训练外的数据)
score=estimater.score(x_test,y_test)
print("准确率为:\n",score)

```

## 7.交叉验证,网格搜索 优化模型

#### 8.1 交叉验证

*   含义:将拿到的训练数据,分为训练集和测试集, 将其中一份作为验证集.
    然后然后经过4组测试,每次更换不同的验证集.取平均值作为最终结果,又称 4折交叉验证

例如:
\| 验证集 | 训练集 | 训练集 | 训练集 | 80% |
\| 训练集 | 验证集 | 训练集 | 训练集 | 78% |
\| 训练集 | 训练集 | 验证集 | 训练集 | 75% |
\| 训练集 | 训练集 | 训练集 | 验证集 | 82% |

每次选取的验证集在不同位置,最终取平均值作为最终结果

*   目的: 为了让被评估的模型更加准确可信 但是不能提高模型预测的准确性,需要使用到`网格搜索`进行优化预测准确率

#### 7.2 网格搜索(Grid Search)

*   超参数定义:
    通常情况下,有很多是需要手动指定的(如K-近邻算法中的K值),这种就叫做超参数.
    但是手动过程繁杂
    所以需要对模型预设几种超参数组合.每组超参数都采用交叉验证来评估.最后选出最优参数组合建立模型

例如:

| K值 | K=3 | K=5 | K=7 |
| -- | --- | --- | --- |
| 模型 | 模型1 | 模型2 | 模型3 |

#### 7.3 交叉验证,网格搜索 (模型选择和调优)API

*   `sklearn.model_selection.GridSearchCV(estimator,param_grid=None,cv=None,n_jobs=3)`
    对估计器的指定参数值进行详尽搜索
    参数:

    estimator: 估算器对象
    param\_grid: 估算器参数 {"n\_neighbors":\[1,3,5],"另外一个超参数":\[xx,xx,xx]}
    cv: 指定几折交叉验证
    n\_jobs: 指定几个CPU来跑  -1表示所有CPU

    fit:输入训练数据
    score: 准确率

    结果返回:
    bestscore\_\_: 在交叉验证中验证的最好结果
    best\_estimator\_: 最好的参数模型
    cvresults:每次交叉验证后的验证集准确率结果和训练集准确率结果

    from sklearn.model\_selection import GridSearchCV

#### 7.5 案例

```
# 交叉验证
# 数据集划分
from  sklearn.datasets import load_iris
# 数据集划分
from sklearn.model_selection import train_test_split
# 交叉验证和网络搜索
from sklearn.model_selection import GridSearchCV
# 数据特征处理-标准化
from sklearn.preprocessing import StandardScaler

# KNN算法
from sklearn.neighbors import KNeighborsClassifier

## 对数据集进行划分
#### 训练集的特征数据x_train,测试集的特征数据x_test,训练集的目标标签(值)y_train,,测试集的目标标签(值)y_test , test_size表示测试集占比 0.2/训练集占比0.8 
x_train,x_test,y_train,y_test=train_test_split(iris.data,iris.target,test_size=0.2,random_state=22)

## 模型标准化
ransfer=StandardScaler()
x_train=ransfer.fit_transform(x_train)
x_test=ransfer.transform(x_test)

# 估算器实例化
estimater=KNeighborsClassifier()

# 模型选择与调优 -网格搜索和交叉验证
## 输入超参数
param_dict={"n_neighbors":[1,3,5]}
## 这里进行交叉验证和网格搜索 CV=交叉验证3次 ,模型调优的网格搜索参数: k值=1,3,5
estimater=GridSearchCV(estimater,param_grid=param_dict,cv=3)

# 模型训练
estimater.fit(x_train,y_train)

## 模型评估
#### 方法1:对比真实值和预测值
y_predict=estimater.predict(x_test);
print("预测结果为:\n",y_predict)
print("对比真实值和预测值:\n",y_predict==y_test)

#### 方法2:直接计算准确率
score=estimater.score(x_test,y_test)
print("准确率为:\n",score)

#### 查看最终选择的结果和交叉验证的结果
print("在交叉验证中验证的最好结果:\n",estimater.best_score_)
print("在交叉验证中验证的最好模型:\n",estimater.best_estimator_)
print("在交叉验证中验证的每次交叉验证的准确性:\n",estimater.cv_results_)


```

比较依据
我们主要关注以下几个指标：

Mean Test Score（平均测试得分）：表示不同 n\_neighbors 下多次折叠的平均得分。
Std Test Score（测试得分的标准差）：表示得分的波动情况。
Rank Test Score（测试得分的排名）：数值越小表示得分越高。

## 8.模型的保存和加载

#### 8.1 导入相关依赖

    pip install joblib

    conda install joblib

#### 8.2 sklearn模型的保存和加载API

     from sklearn.externals import joblib
     * 保存: joblib.dump(estimator,'test.pkl')
     * 加载: estimator=joblib.load(`test.pkl`)

ps: 这个` from sklearn.externals import joblib` API已经废弃
目前需要使用`import joblib`这个新的API

#### 8.3 案例

```
from sklearn.linear_model import LinearRegression
from sklearn.preprocessing import StandardScaler
## 模型保存和加载
import joblib

# 构建数据集
x = [[80, 86], [82, 80], [85, 78], [90, 90], [86, 82], [82, 90], [78, 80]]
y = [84.2, 80.6, 80.1, 90, 83.2, 87.6, 79.4]

# 模型训练
scaler = StandardScaler()
x_scaled = scaler.fit_transform(x)


# estimator = SGDRegressor(max_iter=1000, tol=1e-3, penalty='l2', learning_rate='invscaling', eta0=0.01, random_state=0)
estimator = LinearRegression()
# 使用fit方法进行训练
estimator.fit(x_scaled, y)

## 保存模型
joblib.dump(estimator,'xx.pkl')

## 加载模型
estimator=joblib.load('xx.pkl')

# 输出回归系数和偏置
print("数据集的特征的回归系数:", estimator.coef_)
print("数据集的特征的偏置:", estimator.intercept_)

# 模型预测
## 对新数据点进行相同的标准化处理
new_data_scaled = scaler.transform([[100, 80]])
res = estimator.predict(new_data_scaled)
print("输出平时成绩100分,期末成绩80的最终成绩:", res)

```

