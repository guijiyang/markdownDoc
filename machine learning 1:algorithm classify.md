# 机器学习总结

- [机器学习总结](#%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E6%80%BB%E7%BB%93)
  - [1. 机器学习分类](#1-%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E5%88%86%E7%B1%BB)
    - [1.1. 监督学习](#11-%E7%9B%91%E7%9D%A3%E5%AD%A6%E4%B9%A0)
    - [1.2. 无监督学习](#12-%E6%97%A0%E7%9B%91%E7%9D%A3%E5%AD%A6%E4%B9%A0)
    - [1.3. 半监督学习](#13-%E5%8D%8A%E7%9B%91%E7%9D%A3%E5%AD%A6%E4%B9%A0)
    - [1.4. 强化学习(Reinforcement Learning)](#14-%E5%BC%BA%E5%8C%96%E5%AD%A6%E4%B9%A0reinforcement-learning)
    - [1.5. 总结](#15-%E6%80%BB%E7%BB%93)
  - [2. 数据预处理](#2-%E6%95%B0%E6%8D%AE%E9%A2%84%E5%A4%84%E7%90%86)
    - [2.1 采样](#21-%E9%87%87%E6%A0%B7)
      - [2.1.1 随机采样](#211-%E9%9A%8F%E6%9C%BA%E9%87%87%E6%A0%B7)
      - [2.1.2 系统采样](#212-%E7%B3%BB%E7%BB%9F%E9%87%87%E6%A0%B7)
      - [2.1.2 分层采样](#212-%E5%88%86%E5%B1%82%E9%87%87%E6%A0%B7)
    - [2.2 归一化](#22-%E5%BD%92%E4%B8%80%E5%8C%96)

## 1. 机器学习分类

&emsp;
机器学习算法包括聚类、分类、回归、文本分析等十几种场景的算法，我们讲机器学习算法分为4种——监督学习、半监督学习、无监督学习和增强学习。

### 1.1. 监督学习

通过过往的一些数据的特征以及最终结果来进行训练的方式就是监督学习法。

- *监督学习常见算法*

    |算法场景|算法类型|
    |:-:|:-|
    |分类算法|K近邻、朴素贝叶斯、决策树、随机森林、GBDT 和支持向量机等|
    |回归算法 |逻辑回归、线性回归等|

- *监督学习特点*

    监督学习的一个问题就是获得目标值的成本比较高。例
    如,我们想预测一个电影的好坏,那么在生成训练集的时候
    要依赖于对大量电影的人工标注,这样的人力代价使得监督
    学习在一定程度上是一种成本比较高的学习方法。如何获得
    大量的标记数据一直是监督学习面临的一道难题。

### 1.2. 无监督学习

&emsp;
无监督学习就是指训练样本不依赖于打标数据的机器学习算法。无监督学习主
要是用来解决一些聚类场景的问题,因为当我们的训练数据缺失了目标值之后,能做的事
情就只剩下比对不同样本间的距离关系。常见的无监督学习算法见下表。

| 算法场景 | 算法类型        |
| :------: | --------------- |
| 聚类算法 | K-Means、DBSCAN |
| 推荐算法 | 协同过滤        |

### 1.3. 半监督学习

&emsp;
半监督学习是指通过对样本的部分打标来进行机器学习算法的使用,这种部分打标样本的训练数据的算法应用

### 1.4. 强化学习(Reinforcement Learning)

&emsp;
强调的是系统与外界不断地交互,获得外界的反馈,然后决定自身的行为

### 1.5. 总结

&emsp;
上面就是关于监督学习、无监督学习、半监督学习和强化学习的一些介绍。监督学习
主要解决的是分类和回归的场景,无监督学习主要解决聚类场景,半监督学习解决的是一
些打标数据比较难获得的分类场景,强化学习主要是针对流程中不断需要推理的场景。

|  算法场景  | 算法类型                                           |
| :--------: | -------------------------------------------------- |
|  监督学习  | 逻辑回归、K 近邻、朴素贝叶斯、随机森立、支持向量机 |
| 无监督学习 | K-means、DBSCAN、协同过滤、LDA                     |
| 半监督学习 | 标签传播                                           |
|  强化学习  | 隐马尔可夫                                         |

## 2. 数据预处理

&emsp;
数据预处理是数据挖掘流程的第2步，数据预处理的目的就是把整个数据集调整为对于算法干扰最小的结构，
以便提高最终算法的训练效果。本章主要讲述采样、去噪、归一化和数据过滤。

### 2.1 采样

&emsp;
采样就是按照某种规则从数据集中挑选样本数据。通常的应用场景是数据样本过大，抽取部分样本来训练或者验证，
这样不仅会节约计算资源，在特定条件下也会提升实验效果。

#### 2.1.1 随机采样

&emsp;
随机采样(Random Sampling)是所有采样中最常用的一种，也是最容易实现的采样方法。
实现形式是从被采样数据集中随机地抽取特定数据量，需要指定采样的个数。随机采样分为有放回采样和无放回采样。

示例代码。随机采样的python3.6示例代码如下：

```python
#! /usr/bin/python3
# -*- coding:utf8 -*-

import random

def RandomSampling(dataMat, number):
    try:
        Slice = random.sample(dataMat, number)
        # Iter = iter(Slice)
        # return next(Iter), Slice
        # return Iter.__next__()
        return Slice
    except:
        print('sample larger than population')

def RepetitionRandomSampling(dataMat, number):
    sample = []
    for i in range(number):
        sample.append(dataMat[random.randint(0,len(dataMat)-1)])
    return sample
```

- dataMat是数据集；
- number是采样数；
- RandomSampling是无放回采样；
- RepetitionRandomSampling是放回采样。

#### 2.1.2 系统采样

&emsp;
系统采样(Systematic Sampling)一般情况下是无放回式抽样。系统采样又称为等距采样，
即先将总体的观察单位按某一顺序号分成 n 个部分,再从第一部分随机抽取第 k 号观察单位,
依次用相等间距,从每一部分中各抽取一个观察单位来组成样本。系统采样的使用场景主要是针对按照一定关系排列好的数据。

示例代码如下：

```python
#! /usr/bin/python3
# -*- coding:utf8 -*-

import random
def SystematicSampling(dataMat,number):
    length=len(dataMat)
    k=length/number
    sample=[]
    i=0
    if k>0 :
        while len(sample)!=number:
            sample.append(dataMat[0+i*k])
            i+=1
        return sample
    else :
        return RandomSampling(dataMat,number)
```

- dataMat是数据集；
- number是采样数；
- SystemicSampling是系统采样。

#### 2.1.2 分层采样

&emsp;
分层采样(Stratified Sampling)是先将数据分成若干个类别，再从每一层内随机抽取一定数量的观察样本，然后将这些抽取出来的样本组合起来。分层采样常常被用于在生成训练样本的场景中。因为在监督学习中,通常情况下正负样本的比例是不可控的,当正负
样本的比例过大或者过小的时候,对于训练结果都是有影响的,所以往往我们需要用分层
采样来控制训练样本的正负样本比例。

示例代码如下：

```python
#! /usr/bin/python3
# -*- coding:utf8 -*-

import random
def StratifiedSampling(dataMat1,dataMat2,dataMat3,number):
    length=len(dataMat)
    sample=[]
    num=number/3
    sample.append(RandomSampling(dataMat1,num))
    sample.append(RandomSampling(dataMat2,num))
    sample.append(RandomSampling(dataMat3,num))
    return sample
```

- dataMat1、dataMat2 和 dataMat3 是 3 组数据集;
- number 是采样数;
- StratifiedSampling 是系统采样。

### 2.2 归一化

&emsp;
归一化是指一种简化计算的方式，将数据经过处理之后限定到一定的范围之内，一般都会将数据限定到
[0,1]。数据归一化可以加快算法的收敛速度，而且在后续的数据处理上也会比较方便。另外归一化算法是一种去量纲的行为。
&emsp;
归一化的具体计算方法可以用数据公式来表示:$y=(x-MinValue)/(MaxValue-MinValue)$，这里的$MaxValue$和$MinValue$分别是针对矩阵的每一个字段的最大值和最小值，$x$是字段中的值，$y$为最终归一化结果。