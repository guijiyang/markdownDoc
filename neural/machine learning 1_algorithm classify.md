<!-- @import "my-style.less" -->

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
    - [2.3 去除噪声](#23-%E5%8E%BB%E9%99%A4%E5%99%AA%E5%A3%B0)
    - [2.4 数据过滤](#24-%E6%95%B0%E6%8D%AE%E8%BF%87%E6%BB%A4)
  - [3 特征工程](#3-%E7%89%B9%E5%BE%81%E5%B7%A5%E7%A8%8B)
    - [3.1 特征抽象](#31-%E7%89%B9%E5%BE%81%E6%8A%BD%E8%B1%A1)
    - [3.2 特征重要性评估](#32-%E7%89%B9%E5%BE%81%E9%87%8D%E8%A6%81%E6%80%A7%E8%AF%84%E4%BC%B0)
  - [4 分类算法](#4-%E5%88%86%E7%B1%BB%E7%AE%97%E6%B3%95)
    - [4.1 支持向量机](#41-%E6%94%AF%E6%8C%81%E5%90%91%E9%87%8F%E6%9C%BA)
    - [4.2 随机森林](#42-%E9%9A%8F%E6%9C%BA%E6%A3%AE%E6%9E%97)
  - [5 聚类算法](#5-%E8%81%9A%E7%B1%BB%E7%AE%97%E6%B3%95)

## 1. 机器学习分类

&emsp;机器学习算法包括聚类、分类、回归、文本分析等十几种场景的算法，我们讲机器学习算法分为4种——监督学习、半监督学习、无监督学习和增强学习。

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

&emsp;无监督学习就是指训练样本不依赖于打标数据的机器学习算法。无监督学习主
要是用来解决一些聚类场景的问题,因为当我们的训练数据缺失了目标值之后,能做的事
情就只剩下比对不同样本间的距离关系。常见的无监督学习算法见下表。

|  算法场景 | 算法类型           |
|:-----:|----------------|
|  聚类算法 | K-Means、DBSCAN |
|  推荐算法 | 协同过滤           |

### 1.3. 半监督学习

&emsp;半监督学习是指通过对样本的部分打标来进行机器学习算法的使用,这种部分打标样本的训练数据的算法应用

### 1.4. 强化学习(Reinforcement Learning)

&emsp;强调的是系统与外界不断地交互,获得外界的反馈,然后决定自身的行为

### 1.5. 总结

&emsp;上面就是关于监督学习、无监督学习、半监督学习和强化学习的一些介绍。监督学习
主要解决的是分类和回归的场景,无监督学习主要解决聚类场景,半监督学习解决的是一
些打标数据比较难获得的分类场景,强化学习主要是针对流程中不断需要推理的场景。

|  算法场景 | 算法类型                       |
|:-----:|----------------------------|
|  监督学习 | 逻辑回归、K 近邻、朴素贝叶斯、随机森立、支持向量机 |
| 无监督学习 | K-means、DBSCAN、协同过滤、LDA    |
| 半监督学习 | 标签传播                       |
|  强化学习 | 隐马尔可夫                      |

## 2. 数据预处理

&emsp;数据预处理是数据挖掘流程的第2步，数据预处理的目的就是把整个数据集调整为对于算法干扰最小的结构，
以便提高最终算法的训练效果。本章主要讲述采样、去噪、归一化和数据过滤。

### 2.1 采样

&emsp;采样就是按照某种规则从数据集中挑选样本数据。通常的应用场景是数据样本过大，抽取部分样本来训练或者验证，
这样不仅会节约计算资源，在特定条件下也会提升实验效果。

#### 2.1.1 随机采样

&emsp;随机采样(Random Sampling)是所有采样中最常用的一种，也是最容易实现的采样方法。
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

&emsp;系统采样(Systematic Sampling)一般情况下是无放回式抽样。系统采样又称为等距采样，
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

&emsp;分层采样(Stratified Sampling)是先将数据分成若干个类别，再从每一层内随机抽取一定数量的观察样本，然后将这些抽取出来的样本组合起来。分层采样常常被用于在生成训练样本的场景中。因为在监督学习中,通常情况下正负样本的比例是不可控的,当正负
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

&emsp;归一化是指一种简化计算的方式，将数据经过处理之后限定到一定的范围之内，一般都会将数据限定到[0,1]。数据归一化可以加快算法的收敛速度，而且在后续的数据处理上也会比较方便。另外归一化算法是一种去量纲的行为。

&emsp;归一化的具体计算方法可以用数据公式来表示:

$$
y=(x-MinValue)/(MaxValue-MinValue)
$$

&emsp;这里的 $MaxValue$ 和 $MinValue$ 分别是针对矩阵的每一个字段的最大值和最小值， $x$ 是字段中的值， $y$ 为最终归一化结果。

- >示例代码：width函数用来获取矩阵的字段数量；AutoNorm函数对矩阵执行归一化。

```python
def width(lst):
    i = 0
    for j in lst[0]:
        i = i+1
    return i

def AutoNorm(mat):
    n=len(mat)
    m=Width(mat)
    MinNum=[999999999]*m
    MaxNum=[0]*m

    for col in mat:
        for row in range(0,m):
            if col[row] > MaxNum[row]:
                MaxNum[row] = col[row];

    for col in mat:
        for row in range(0,m):
            if col[row] < MinNum[row]:
                MinNum[row]=col[row]

    section = list(map(lambda x: x[0]-x[1], zip(MaxNum,MinNum)))
    NormMat=[]

    for col in mat:
        distance=list(map(lambda x: x[0]-x[1], zip(col, MinNum)))
        value=list(map(lambda x: x[0]/x[1], zip(distance, section)))
        NormMat.append(value)
    return NormMat
```

### 2.3 去除噪声

&emsp;噪声数据究竟对于模型训练有什么影响呢?因为很多算法,特别是线性算法,都是通过迭代来获取最优解的,如果数据集中含有大量的噪声数据,将会大大影响数据的收敛速率,甚至对于训练所生成模型的准确度也会有很大的负面作用。
&emsp;根据不同的业务场景，有不同的处理方法。常见的一个方法为正态分布3$\sigma$方法。正态分布可以表示成一种概率密度函数，如下所示:$$f(x)=\frac{1}{\sqrt{2\pi\sigma}}exp\Big(-\frac{(x-\mu)^2}{2\sigma^2}\Big)$$其中$\sigma$可以表示称数据集的方差，$\mu$代表数据集的均值，$x$代表数据集的数据。
&emsp;正态分布具备这样的特点:x 落在(μ−3σ,μ+3σ)以外的概率小于千分之三。根据这种特点,我们可以通过计算数据集的方差,把 3 倍方差之外的点设想为噪声数据来排除,这是比较常规的去噪手段。

- 示例代码：GetAverage 函数返回均值;width 函数获取矩阵的宽度;GetVar 函数获取方差;DenoisMat 函数用于去噪。

```python
def GetAverage(mat):
    n = len(mat)
    m = width(mat)
    num = [0]*m
    for j in range(0, m):
        for i in mat:
            num[j] = num[j]+i[j]
        num[j] = num[j]/n
    return num

def GetVar(average, mat):
    ListMat = []
    for i in mat:
        ListMat.append(list(map(lambda x: x[0]-x[1], zip(average, i))))

    n = len(ListMat)
    m = width(ListMat)
    num = [0]*m
    for j in range(0, m):
        for i in ListMat:
            num[j] = num[j] + (i[j]*i[j])
        num[j] = num[j]/n
    return num


def DenoisMat(mat):
    average = GetAverage(mat)
    variance = GetVar(average, mat)
    section = list(map(lambda x: x[0]+x[1], zip(average, variance)))
    n = len(mat)
    m = width(mat)
    num = [0]*m
    denoisMat = []
    for i in mat:
        for j in range(0, m):
            if i[j] > section[j]:
                i[j] = section[j]
        denoisMat.append(i)
    return denoisMat

```

### 2.4 数据过滤

&emsp;数据挖掘用到的数据来源于用户行为日志或是日常记录等,在很多时候不可以把全部数据进行训练。通常对于同一份数据,如果想做不同目的的挖掘,需要做不同方式的处理,数据过滤往往是数据前期处理的重要一环。例如我们有一组数据,里面有一个字段对于结果没有任何的意义，则需要过滤掉这个字段。

## 3 特征工程

&emsp;前面数据预处理主要是一些数据的抽取（Extract）、清洗（cleaning）、转换（transform）、装载(Load)工作。特征工程也可以从几方面来看，分别是特征抽象、特征重要性评估、特征衍生和特征降维几个方面。

### 3.1 特征抽象

&emsp;特征抽象是指将源数据抽象成算法可以理解的数据。这个概念可能听起来比较难以理解,我们举一些例子来时行说明。首先思考一下什么样的数据是算法可以理解的。目前的机器学习算法主要是对一些数据进行矩阵运算或者概率运算来得到结论,也就是说算法的入参通常来讲应该是一组可以表达数据某类特性的数字,基于这个理论,我们期望的数据应该见表 3-1,所有属性都通过数字来表示。

<font color=Magenta><center>表 3-1 理想数据</center></font>

| 用 户 昵 称 | 用户ID | 用户购买次数 | 用 户 性 别 | 是 否 购 买 |
|---------|------|--------|---------|---------|
| 星大大     | 4212 | 3      | 1       | 1       |
| 琪琪      | 2141 | 2      | 0       | 0       |
&emsp;但是实际上,在真实的业务场景下,做数据采集的跟做分析的往往是不同的人,我们
拿到手的数据很有可能是这样的,见表 3-2,是一串文本。

<font color=Magenta><center>表 3-2 实际数据</center></font>

| 用 户 昵 称 | 用户 ID | 用户购买次数   | 用 户 性 别 | 是 否 购 买 |
|---------|-------|----------|---------|---------|
| 星大大     | 4212  | 用户购买了好多次 | 好像是女的   | 忘了买没买   |
| 琪琪      | 2141  | 用户不怎么买   | 是女的     | 买了      |
&emsp;这时怎么利用这些非结构化数据进行分析？这就需要我们对数据进行抽象。抽象的过程没有现成的公式或者是工具，往往要根据用户的主观判断来实现。特征抽象需要具备两方面能力,一方面是需要进行大量的相关工作来积累经验,另一方面需要对特征抽象的数据的业务场景足够了解。

### 3.2 特征重要性评估

## 4 分类算法

分类算法是用来解决分类问题的算法。分类算法属于监督学习，通常是根据打标好的训练数据建立模型。

### 4.1 支持向量机

以二维数据为例,实心和空心的两组数据可以通过一条直线进行分类。这条用于分类的直线就叫作分类机。这个分类机就是支持向量机中这个“机”的含义。在二分类的这条线上下,都有每个类别跟分类器间隔最近的一些点,这些点被标记出来。如果这些点的位置发生了变化,那么分类机的位置也相应地会发生变化,也就是说这些间隔点支持了分类机,这些间隔点就是“支持向量”。到了这一步,我们通过名字已经知道了这个算法的组成和含义,就是支持向量和分类机。下面只要保证每一个类别的这些支持向量跟分类机的几何距离最大,就可以保证分类的准确度。

- SVM(support vector machine)支持向量机新颖的地方是基于小样本学习的算法，SVM算法中我们通过找到支持向量(距离分类机最近的类别的点)来进行分类，跟传统的逻辑回归算法这种利用全样本点的算法有所不同。
- SVM在分类的时候引入了最大边界的思想。当确定了支持向量后，找寻正负支持向量间距最大的的平面。
- 对于非线性分类，支持向量机通过讲低纬度的数据映射到高纬度的空间去求解。
- 由于SVM算法是通过小样本的支持向量来分类，这会给结果造成很大的鲁棒性，如果只是改变了样本数据集，但是支持向量没变的情况下，这时的分类机改变不明显。
- SVM算法的缺陷是就是处理多分类的问题不如逻辑回归和树状结构灵活。SVM主要还是用于二分类场景。

### 4.2 随机森林

随机森林是由多个决策树组成的分类器，是一种监督学习方法。每个决策树都是一个弱分类器，分类结果由这些决策树决定。
决策树是一种常见的监督学习方法，呈树状结构，每一个节点都是一个属性(特征)，每一个分支代表一个输出，最终的叶节点表示分类结果，如下图所示是通过监督学习得到的一个决策树分类器。

![tree classify](/home/guijiyang/Code/markdown/img/tree\ structure\ classify.png "tree-classify")

决策树通常由二叉树或者多叉树组成,由上面的图可以看出，越上面的树节点对结果影响越明显。

了解了决策树的概念后，可以知道随机森林的图形如下图所示。

![tree](/home/guijiyang/Code/markdown/img/tree\ structure.png)

**需要注意的地方**:

- 每个决策树的特征是由抽样生成的。例如训练集一共有 M 个特征,那么每次有放回地抽取 m 个特征作为决策树的训练特征集,并且 m<M。
- 每棵树的训练数据也是由抽样生成的，如果训练集有 N 个训练数据,那么每棵树在训练模型的时候会抽样 n 个样例数据进行训练,这个抽样是有放回的抽样。
- 决策树一般都选用二叉决策树,最终的结果通过所有的决策树投票来决定。投票的方式有很多种,通常可以通过比对不同分类结果的树的数量,选取数量多的一方作为最终的分类结果。

## 5 聚类算法

聚类算法就是将一组数据按照类别归类。聚类算法和分类算法看似都是将数据按照一定标准进行区分。但是分类算法属于监督学习，需要通过打标的数据建立模型，然后对未知数据进行预测。聚类算法属于无监督学习，就是没有打标好的数据场景,如在用户画像的时候,我们有一组数据表示的是用户的年龄、收入和兴趣等数据,这样就可以通过聚类算法把相似度高的用户分到同一个类别中。

聚类算法根据计算方式的不同可以分为基于距离的算法和基于密度的算法。