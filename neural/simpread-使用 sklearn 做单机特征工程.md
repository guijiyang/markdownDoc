> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 https://www.cnblogs.com/jasonfreak/p/5448385.html

目录
==

1 特征工程是什么？  
2 数据预处理  
　　2.1 无量纲化  
　　　　2.1.1 标准化  
　　　　2.1.2 区间缩放法  
　　　　2.1.3 标准化与归一化的区别  
　　2.2 对定量特征二值化  
　　2.3 对定性特征哑编码  
　　2.4 缺失值计算  
　　2.5 数据变换  
　　2.6 回顾  
3 特征选择  
　　3.1 Filter  
　　　　3.1.1 方差选择法  
　　　　3.1.2 相关系数法  
　　　　3.1.3 卡方检验  
　　　　3.1.4 互信息法  
　　3.2 Wrapper  
　　　　3.2.1 递归特征消除法  
　　3.3 Embedded  
　　　　3.3.1 基于惩罚项的特征选择法  
　　　　3.3.2 基于树模型的特征选择法  
　　3.4 回顾  
4 降维  
　　4.1 主成分分析法（PCA）  
　　4.2 线性判别分析法（LDA）  
　　4.3 回顾  
5 总结  
6 参考资料

1 特征工程是什么？
==========

　　有这么一句话在业界广泛流传：数据和特征决定了机器学习的上限，而模型和算法只是逼近这个上限而已。那特征工程到底是什么呢？顾名思义，其本质是一项工程活动，目的是最大限度地从原始数据中提取特征以供算法和模型使用。通过总结和归纳，人们认为特征工程包括以下方面：

![](https://images2015.cnblogs.com/blog/927391/201604/927391-20160430145122660-830141495.jpg)

　　特征处理是特征工程的核心部分，sklearn 提供了较为完整的特征处理方法，包括数据预处理，特征选择，降维等。首次接触到 sklearn，通常会被其丰富且方便的算法模型库吸引，但是这里介绍的特征处理库也十分强大！

　　本文中使用 sklearn 中的 [IRIS（鸢尾花）数据集](http://scikit-learn.org/stable/modules/generated/sklearn.datasets.load_iris.html#sklearn.datasets.load_iris)来对特征处理功能进行说明。IRIS 数据集由 Fisher 在 1936 年整理，包含 4 个特征（Sepal.Length（花萼长度）、Sepal.Width（花萼宽度）、Petal.Length（花瓣长度）、Petal.Width（花瓣宽度）），特征值都为正浮点数，单位为厘米。目标值为鸢尾花的分类（Iris Setosa（山鸢尾）、Iris Versicolour（杂色鸢尾），Iris Virginica（维吉尼亚鸢尾））。导入 IRIS 数据集的代码如下：

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

```
 1 from sklearn.datasets import load_iris
 2 
 3 #导入IRIS数据集
 4 iris = load_iris()
 5 
 6 #特征矩阵
 7 iris.data
 8 
 9 #目标向量
10 iris.target

```

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

2 数据预处理
=======

　　通过特征提取，我们能得到未经处理的特征，这时的特征可能有以下问题：

*   不属于同一量纲：即特征的规格不一样，不能够放在一起比较。无量纲化可以解决这一问题。
*   信息冗余：对于某些定量特征，其包含的有效信息为区间划分，例如学习成绩，假若只关心 “及格” 或不 “及格”，那么需要将定量的考分，转换成“1” 和“0”表示及格和未及格。二值化可以解决这一问题。
*   定性特征不能直接使用：某些机器学习算法和模型只能接受定量特征的输入，那么需要将定性特征转换为定量特征。最简单的方式是为每一种定性值指定一个定量值，但是这种方式过于灵活，增加了调参的工作。[通常使用哑编码的方式将定性特征转换为定量特征](http://www.ats.ucla.edu/stat/mult_pkg/faq/general/dummy.htm)：假设有 N 种定性值，则将这一个特征扩展为 N 种特征，当原始特征值为第 i 种定性值时，第 i 个扩展特征赋值为 1，其他扩展特征赋值为 0。哑编码的方式相比直接指定的方式，不用增加调参的工作，对于线性模型来说，使用哑编码后的特征可达到非线性的效果。
*   存在缺失值：缺失值需要补充。
*   信息利用率低：不同的机器学习算法和模型对数据中信息的利用是不同的，之前提到在线性模型中，使用对定性特征哑编码可以达到非线性的效果。类似地，对定量变量多项式化，或者进行其他的转换，都能达到非线性的效果。

　　我们使用 sklearn 中的 preproccessing 库来进行数据预处理，可以覆盖以上问题的解决方案。

2.1 无量纲化
--------

　　无量纲化使不同规格的数据转换到同一规格。常见的无量纲化方法有标准化和区间缩放法。标准化的前提是特征值服从正态分布，标准化后，其转换成标准正态分布。区间缩放法利用了边界值信息，将特征的取值区间缩放到某个特点的范围，例如 [0, 1] 等。

### 2.1.1 标准化

　　标准化需要计算特征的均值和标准差，公式表达为：

![](https://images2015.cnblogs.com/blog/927391/201605/927391-20160502113957732-1062097580.png)

　　使用 preproccessing 库的 StandardScaler 类对数据进行标准化的代码如下：

```
1 from sklearn.preprocessing import StandardScaler
2 
3 #标准化，返回值为标准化后的数据
4 StandardScaler().fit_transform(iris.data)

```

### 2.1.2 区间缩放法

　　区间缩放法的思路有多种，常见的一种为利用两个最值进行缩放，公式表达为：

![](https://images2015.cnblogs.com/blog/927391/201605/927391-20160502113301013-1555489078.png)

　　使用 preproccessing 库的 MinMaxScaler 类对数据进行区间缩放的代码如下：

```
1 from sklearn.preprocessing import MinMaxScaler
2 
3 #区间缩放，返回值为缩放到[0, 1]区间的数据
4 MinMaxScaler().fit_transform(iris.data)

```

### 2.1.3 标准化与归一化的区别

　　简单来说，标准化是依照特征矩阵的列处理数据，其通过求 z-score 的方法，将样本的特征值转换到同一量纲下。归一化是依照特征矩阵的行处理数据，其目的在于样本向量在点乘运算或其他核函数计算相似性时，拥有统一的标准，也就是说都转化为 “单位向量”。规则为 l2 的归一化公式如下：

![](https://images2015.cnblogs.com/blog/927391/201607/927391-20160719002904919-1602367496.png)

　　使用 preproccessing 库的 Normalizer 类对数据进行归一化的代码如下：

```
1 from sklearn.preprocessing import Normalizer
2 
3 #归一化，返回值为归一化后的数据
4 Normalizer().fit_transform(iris.data)

```

2.2 对定量特征二值化
------------

　　定量特征二值化的核心在于设定一个阈值，大于阈值的赋值为 1，小于等于阈值的赋值为 0，公式表达如下：

![](https://images2015.cnblogs.com/blog/927391/201605/927391-20160502115121216-456946808.png)

　　使用 preproccessing 库的 Binarizer 类对数据进行二值化的代码如下：

```
1 from sklearn.preprocessing import Binarizer
2 
3 #二值化，阈值设置为3，返回值为二值化后的数据
4 Binarizer(threshold=3).fit_transform(iris.data)

```

2.3 对定性特征哑编码
------------

　　由于 IRIS 数据集的特征皆为定量特征，故使用其目标值进行哑编码（实际上是不需要的）。使用 preproccessing 库的 OneHotEncoder 类对数据进行哑编码的代码如下：

```
1 from sklearn.preprocessing import OneHotEncoder
2 
3 #哑编码，对IRIS数据集的目标值，返回值为哑编码后的数据
4 OneHotEncoder().fit_transform(iris.target.reshape((-1,1)))

```

2.4 缺失值计算
---------

　　由于 IRIS 数据集没有缺失值，故对数据集新增一个样本，4 个特征均赋值为 NaN，表示数据缺失。使用 preproccessing 库的 Imputer 类对数据进行缺失值计算的代码如下：

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

```
1 from numpy import vstack, array, nan
2 from sklearn.preprocessing import Imputer
3 
4 #缺失值计算，返回值为计算缺失值后的数据
5 #参数missing_value为缺失值的表示形式，默认为NaN
6 #参数strategy为缺失值填充方式，默认为mean（均值）
7 Imputer().fit_transform(vstack((array([nan, nan, nan, nan]), iris.data)))

```

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

2.5 数据变换
--------

　　常见的数据变换有基于多项式的、基于指数函数的、基于对数函数的。4 个特征，度为 2 的多项式转换公式如下：

![](https://images2015.cnblogs.com/blog/927391/201605/927391-20160502134944451-270339895.png)

　　使用 preproccessing 库的 PolynomialFeatures 类对数据进行多项式转换的代码如下：

```
1 from sklearn.preprocessing import PolynomialFeatures
2 
3 #多项式转换
4 #参数degree为度，默认值为2
5 PolynomialFeatures().fit_transform(iris.data)

```

　　基于单变元函数的数据变换可以使用一个统一的方式完成，使用 preproccessing 库的 FunctionTransformer 对数据进行对数函数转换的代码如下：

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

```
1 from numpy import log1p
2 from sklearn.preprocessing import FunctionTransformer
3 
4 #自定义转换函数为对数函数的数据变换
5 #第一个参数是单变元函数
6 FunctionTransformer(log1p).fit_transform(iris.data)

```

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

2.6 回顾
------

<table border="0" align="center"><tbody><tr><td>类</td><td>功能</td><td>说明</td></tr><tr><td>StandardScaler</td><td>无量纲化</td><td>标准化，基于特征矩阵的列，将特征值转换至服从标准正态分布</td></tr><tr><td>MinMaxScaler</td><td>无量纲化</td><td>区间缩放，基于最大最小值，将特征值转换到 [0, 1] 区间上</td></tr><tr><td>Normalizer</td><td>归一化</td><td>基于特征矩阵的行，将样本向量转换为 “单位向量”</td></tr><tr><td>Binarizer</td><td>二值化</td><td>基于给定阈值，将定量特征按阈值划分</td></tr><tr><td>OneHotEncoder</td><td>哑编码</td><td>将定性数据编码为定量数据</td></tr><tr><td>Imputer</td><td>缺失值计算</td><td>计算缺失值，缺失值可填充为均值等</td></tr><tr><td>PolynomialFeatures</td><td>多项式数据转换</td><td>多项式数据转换</td></tr><tr><td>FunctionTransformer</td><td>自定义单元数据转换</td><td>使用单变元的函数来转换数据</td></tr></tbody></table>

3 特征选择
======

　　当数据预处理完成后，我们需要选择有意义的特征输入机器学习的算法和模型进行训练。通常来说，从两个方面考虑来选择特征：

*   特征是否发散：如果一个特征不发散，例如方差接近于 0，也就是说样本在这个特征上基本上没有差异，这个特征对于样本的区分并没有什么用。
*   特征与目标的相关性：这点比较显见，与目标相关性高的特征，应当优选选择。除方差法外，本文介绍的其他方法均从相关性考虑。

　　根据特征选择的形式又可以将特征选择方法分为 3 种：

*   Filter：过滤法，按照发散性或者相关性对各个特征进行评分，设定阈值或者待选择阈值的个数，选择特征。
*   Wrapper：包装法，根据目标函数（通常是预测效果评分），每次选择若干特征，或者排除若干特征。
*   Embedded：嵌入法，先使用某些机器学习的算法和模型进行训练，得到各个特征的权值系数，根据系数从大到小选择特征。类似于 Filter 方法，但是是通过训练来确定特征的优劣。

　　我们使用 sklearn 中的 feature_selection 库来进行特征选择。

3.1 Filter
----------

### 3.1.1 方差选择法

　　使用方差选择法，先要计算各个特征的方差，然后根据阈值，选择方差大于阈值的特征。使用 feature_selection 库的 VarianceThreshold 类来选择特征的代码如下：

```
1 from sklearn.feature_selection import VarianceThreshold
2 
3 #方差选择法，返回值为特征选择后的数据
4 #参数threshold为方差的阈值
5 VarianceThreshold(threshold=3).fit_transform(iris.data)

```

### 3.1.2 相关系数法

　　使用相关系数法，先要计算各个特征对目标值的相关系数以及相关系数的 P 值。用 feature_selection 库的 SelectKBest 类结合相关系数来选择特征的代码如下：

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

```
1 from sklearn.feature_selection import SelectKBest
2 from scipy.stats import pearsonr
3 
4 #选择K个最好的特征，返回选择特征后的数据
5 #第一个参数为计算评估特征是否好的函数，该函数输入特征矩阵和目标向量，输出二元组（评分，P值）的数组，数组第i项为第i个特征的评分和P值。在此定义为计算相关系数
6 #参数k为选择的特征个数
7 SelectKBest(lambda X, Y: array(map(lambda x:pearsonr(x, Y), X.T)).T, k=2).fit_transform(iris.data, iris.target)

```

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

### 3.1.3 卡方检验

　　经典的卡方检验是检验定性自变量对定性因变量的相关性。假设自变量有 N 种取值，因变量有 M 种取值，考虑自变量等于 i 且因变量等于 j 的样本频数的观察值与期望的差距，构建统计量：

![](https://images2015.cnblogs.com/blog/927391/201605/927391-20160502144243326-2086446424.png)

　　[这个统计量的含义简而言之就是自变量对因变量的相关性](http://wiki.mbalib.com/wiki/%E5%8D%A1%E6%96%B9%E6%A3%80%E9%AA%8C)。用 feature_selection 库的 SelectKBest 类结合卡方检验来选择特征的代码如下：

```
1 from sklearn.feature_selection import SelectKBest
2 from sklearn.feature_selection import chi2
3 
4 #选择K个最好的特征，返回选择特征后的数据
5 SelectKBest(chi2, k=2).fit_transform(iris.data, iris.target)

```

### 3.1.4 互信息法

　　经典的互信息也是评价定性自变量对定性因变量的相关性的，互信息计算公式如下：

![](https://images2015.cnblogs.com/blog/927391/201605/927391-20160502145723247-1184422794.png)

　　为了处理定量数据，最大信息系数法被提出，使用 feature_selection 库的 SelectKBest 类结合最大信息系数法来选择特征的代码如下：

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

```
 1 from sklearn.feature_selection import SelectKBest
 2 from minepy import MINE
 3 
 4 #由于MINE的设计不是函数式的，定义mic方法将其为函数式的，返回一个二元组，二元组的第2项设置成固定的P值0.5
 5 def mic(x, y):
 6     m = MINE()
 7     m.compute_score(x, y)
 8     return (m.mic(), 0.5)
 9 
10 #选择K个最好的特征，返回特征选择后的数据
11 SelectKBest(lambda X, Y: array(map(lambda x:mic(x, Y), X.T)).T, k=2).fit_transform(iris.data, iris.target)

```

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

3.2 Wrapper
-----------

### 3.2.1 递归特征消除法

　　递归消除特征法使用一个基模型来进行多轮训练，每轮训练后，消除若干权值系数的特征，再基于新的特征集进行下一轮训练。使用 feature_selection 库的 RFE 类来选择特征的代码如下：

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

```
1 from sklearn.feature_selection import RFE
2 from sklearn.linear_model import LogisticRegression
3 
4 #递归特征消除法，返回特征选择后的数据
5 #参数estimator为基模型
6 #参数n_features_to_select为选择的特征个数
7 RFE(estimator=LogisticRegression(), n_features_to_select=2).fit_transform(iris.data, iris.target)

```

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

3.3 Embedded
------------

### 3.3.1 基于惩罚项的特征选择法

　　使用带惩罚项的基模型，除了筛选出特征外，同时也进行了降维。使用 feature_selection 库的 SelectFromModel 类结合带 L1 惩罚项的逻辑回归模型，来选择特征的代码如下：

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

```
1 from sklearn.feature_selection import SelectFromModel
2 from sklearn.linear_model import LogisticRegression
3 
4 #带L1惩罚项的逻辑回归作为基模型的特征选择
5 SelectFromModel(LogisticRegression(penalty="l1", C=0.1)).fit_transform(iris.data, iris.target)

```

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

　　[L1 惩罚项降维的原理在于保留多个对目标值具有同等相关性的特征中的一个](http://www.zhihu.com/question/28641663/answer/41653367)，所以没选到的特征不代表不重要。故，可结合 L2 惩罚项来优化。具体操作为：若一个特征在 L1 中的权值为 1，选择在 L2 中权值差别不大且在 L1 中权值为 0 的特征构成同类集合，将这一集合中的特征平分 L1 中的权值，故需要构建一个新的逻辑回归模型：

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)![](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

```
 1 from sklearn.linear_model import LogisticRegression
 2 
 3 class LR(LogisticRegression):
 4     def __init__(self, threshold=0.01, dual=False, tol=1e-4, C=1.0,
 5                  fit_intercept=True, intercept_scaling=1, class_weight=None,
 6                  random_state=None, solver='liblinear', max_iter=100,
 7                  multi_class='ovr', verbose=0, warm_start=False, n_jobs=1):
 8 
 9         #权值相近的阈值
10         self.threshold = threshold
11         LogisticRegression.__init__(self, penalty='l1', dual=dual, tol=tol, C=C,
12                  fit_intercept=fit_intercept, intercept_scaling=intercept_scaling, class_weight=class_weight,
13                  random_state=random_state, solver=solver, max_iter=max_iter,
14                  multi_class=multi_class, verbose=verbose, warm_start=warm_start, n_jobs=n_jobs)
15         #使用同样的参数创建L2逻辑回归
16         self.l2 = LogisticRegression(penalty='l2', dual=dual, tol=tol, C=C, fit_intercept=fit_intercept, intercept_scaling=intercept_scaling, class_weight = class_weight, random_state=random_state, solver=solver, max_iter=max_iter, multi_class=multi_class, verbose=verbose, warm_start=warm_start, n_jobs=n_jobs)
17 
18     def fit(self, X, y, sample_weight=None):
19         #训练L1逻辑回归
20         super(LR, self).fit(X, y, sample_weight=sample_weight)
21         self.coef_old_ = self.coef_.copy()
22         #训练L2逻辑回归
23         self.l2.fit(X, y, sample_weight=sample_weight)
24 
25         cntOfRow, cntOfCol = self.coef_.shape
26         #权值系数矩阵的行数对应目标值的种类数目
27         for i in range(cntOfRow):
28             for j in range(cntOfCol):
29                 coef = self.coef_[i][j]
30                 #L1逻辑回归的权值系数不为0
31                 if coef != 0:
32                     idx = [j]
33                     #对应在L2逻辑回归中的权值系数
34                     coef1 = self.l2.coef_[i][j]
35                     for k in range(cntOfCol):
36                         coef2 = self.l2.coef_[i][k]
37                         #在L2逻辑回归中，权值系数之差小于设定的阈值，且在L1中对应的权值为0
38                         if abs(coef1-coef2) < self.threshold and j != k and self.coef_[i][k] == 0:
39                             idx.append(k)
40                     #计算这一类特征的权值系数均值
41                     mean = coef / len(idx)
42                     self.coef_[i][idx] = mean
43         return self

```

View Code

　　使用 feature_selection 库的 SelectFromModel 类结合带 L1 以及 L2 惩罚项的逻辑回归模型，来选择特征的代码如下：

```
1 from sklearn.feature_selection import SelectFromModel
2 
3 #带L1和L2惩罚项的逻辑回归作为基模型的特征选择
4 #参数threshold为权值系数之差的阈值
5 SelectFromModel(LR(threshold=0.5, C=0.1)).fit_transform(iris.data, iris.target)

```

### 3.3.2 基于树模型的特征选择法

　　树模型中 GBDT 也可用来作为基模型进行特征选择，使用 feature_selection 库的 SelectFromModel 类结合 GBDT 模型，来选择特征的代码如下：

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

```
1 from sklearn.feature_selection import SelectFromModel
2 from sklearn.ensemble import GradientBoostingClassifier
3 
4 #GBDT作为基模型的特征选择
5 SelectFromModel(GradientBoostingClassifier()).fit_transform(iris.data, iris.target)

```

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

3.4 回顾
------

<table border="0" align="center"><tbody><tr><td>类</td><td>所属方式</td><td>说明</td></tr><tr><td>VarianceThreshold</td><td>Filter</td><td>方差选择法</td></tr><tr><td>SelectKBest</td><td>Filter</td><td>可选关联系数、卡方校验、最大信息系数作为得分计算的方法</td></tr><tr><td>RFE</td><td>Wrapper</td><td>递归地训练基模型，将权值系数较小的特征从特征集合中消除</td></tr><tr><td>SelectFromModel</td><td>Embedded</td><td>训练基模型，选择权值系数较高的特征</td></tr></tbody></table>

4 降维
====

　　当特征选择完成后，可以直接训练模型了，但是可能由于特征矩阵过大，导致计算量大，训练时间长的问题，因此降低特征矩阵维度也是必不可少的。常见的降维方法除了以上提到的基于 L1 惩罚项的模型以外，另外还有主成分分析法（PCA）和线性判别分析（LDA），线性判别分析本身也是一个分类模型。PCA 和 LDA 有很多的相似点，其本质是要将原始的样本映射到维度更低的样本空间中，但是 PCA 和 LDA 的映射目标不一样：[PCA 是为了让映射后的样本具有最大的发散性；而 LDA 是为了让映射后的样本有最好的分类性能](http://www.cnblogs.com/LeftNotEasy/archive/2011/01/08/lda-and-pca-machine-learning.html)。所以说 PCA 是一种无监督的降维方法，而 LDA 是一种有监督的降维方法。

4.1 主成分分析法（PCA）
---------------

　　使用 decomposition 库的 PCA 类选择特征的代码如下：

```
1 from sklearn.decomposition import PCA
2 
3 #主成分分析法，返回降维后的数据
4 #参数n_components为主成分数目
5 PCA(n_components=2).fit_transform(iris.data)

```

4.2 线性判别分析法（LDA）
----------------

　　使用 lda 库的 LDA 类选择特征的代码如下：

```
1 from sklearn.lda import LDA
2 
3 #线性判别分析法，返回降维后的数据
4 #参数n_components为降维后的维数
5 LDA(n_components=2).fit_transform(iris.data, iris.target)

```

4.3 回顾
------

<table border="0" align="center"><tbody><tr><td>库</td><td>类</td><td>说明</td></tr><tr><td>decomposition</td><td>PCA</td><td>主成分分析法</td></tr><tr><td>lda</td><td>LDA</td><td>线性判别分析法</td></tr></tbody></table>

5 总结
====

　　再让我们回归一下本文开始的特征工程的思维导图，我们可以使用 sklearn 完成几乎所有特征处理的工作，而且不管是数据预处理，还是特征选择，抑或降维，它们都是通过某个类的方法 fit_transform 完成的，fit_transform 要不只带一个参数：特征矩阵，要不带两个参数：特征矩阵加目标向量。这些难道都是巧合吗？还是故意设计成这样？方法 fit_transform 中有 fit 这一单词，它和训练模型的 fit 方法有关联吗？接下来，我将在[《使用 sklearn 优雅地进行数据挖掘》](http://www.cnblogs.com/jasonfreak/p/5448462.html)中阐述其中的奥妙！

6 参考资料
======

1.  [FAQ: What is dummy coding?](http://www.ats.ucla.edu/stat/mult_pkg/faq/general/dummy.htm)
2.  [IRIS（鸢尾花）数据集](http://scikit-learn.org/stable/modules/generated/sklearn.datasets.load_iris.html#sklearn.datasets.load_iris)
3.  [卡方检验](http://wiki.mbalib.com/wiki/%E5%8D%A1%E6%96%B9%E6%A3%80%E9%AA%8C)
4.  [干货：结合 Scikit-learn 介绍几种常用的特征选择方法](http://dataunion.org/14072.html)
5.  [机器学习中，有哪些特征选择的工程方法？](http://www.zhihu.com/question/28641663/answer/41653367)
6.  [机器学习中的数学 (4)- 线性判别分析（LDA）, 主成分分析 (PCA)](http://www.cnblogs.com/LeftNotEasy/archive/2011/01/08/lda-and-pca-machine-learning.html)