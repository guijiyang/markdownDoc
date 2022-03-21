<!-- @import "../my-style.less" -->
<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [高斯混合模型](#高斯混合模型)
  - [1. 高斯分布](#1-高斯分布)
  - [2. 高斯混合模型 (Gaussian Mixture Model)](#2-高斯混合模型-gaussian-mixture-model)
  - [3. 隐变量 & 完全数据](#3-隐变量-完全数据)
  - [4. 后验概率 & 响应度](#4-后验概率-响应度)
  - [5. 对数似然函数 & 最大似然估计](#5-对数似然函数-最大似然估计)
  - [7. 高斯混合模型的 EM 算法](#7-高斯混合模型的-em-算法)
  - [8. EM 算法的一般形式](#8-em-算法的一般形式)
  - [9. 换个角度看高斯混合模型的 EM 算法](#9-换个角度看高斯混合模型的-em-算法)

<!-- /code_chunk_output -->

## 高斯混合模型
### 1. 高斯分布
-------

高斯分布 (Gaussian distribution)，也称为正态分布 (normal distribution)，是一种常用的连续变量分布的模型。若单个随机变量 $x$ 服从均值为 $\mu$ ，方差为 $\sigma^2$ 的高斯分布，记为 $x\sim N(\mu,\sigma^2)$ ，则其概率密度函数为：
$$p(x|\mu,\sigma^2)=\frac{1}{(2\pi\sigma^2)^{1/2}}\exp\Big\lbrace-\frac{1}{2\sigma^2}(x-\mu)^2 \Big\rbrace$$

对于一个 $D$ 维的向量 $x$ ，若其各元素服从均值为向量 $\mu$ ，协方差矩阵为 $\Sigma$ 的多元高斯分布，记为 $x\sim N(\mu,\Sigma)$ ，则概率密度为：

$$p(x|\mu,\sigma^2)=\frac{1}{(2\pi)^{D/2}|\Sigma|^{1/2}}\exp \Big\lbrace -\frac{1}{2}(x-\mu)^T\Sigma^{-1}(x-\mu)\Big\rbrace\tag{1}$$

其中 $\mu$ 为 $D$ 维均值向量，$\Sigma$为 $D\times D$ 的协方差矩阵， $|\Sigma|$ 表示 $\Sigma$ 的行列式。
(1) 式中，指数部分的二次型 $\Delta^2=(x-\mu)^T\Sigma^{-1}(x-\mu)$ 称为 $x$ 到 $\mu$ 的马哈拉诺比斯距离 (马氏距离，[Mahalanobis distance](https://link.zhihu.com/?target=https%3A//en.wikipedia.org/wiki/Mahalanobis_distance))；当 $\Sigma$ 为单位矩阵时退化为欧几里得距离 (Euclidean distance)。多元高斯分布密度函数的等高线即 $\Delta^2$ 为常数时 $x$ 的方程，是椭球方程 ([Ellipsoid - Wikipedia](https://link.zhihu.com/?target=https%3A//en.wikipedia.org/wiki/Ellipsoid))。

### 2. 高斯混合模型 (Gaussian Mixture Model)
----------------------------------

多个高斯分布的线性叠加能拟合非常复杂的密度函数；通过足够多的高斯分布叠加，并调节它们的均值，协方差矩阵，以及线性组合的系数，可以精确地逼近任意连续密度 ([1], Section 2.3.9, p111)。

我们考虑 $K$ 个高斯分布的线性叠加，这个高斯混合分布 (Gaussian mixture distiburion) 的概率密度函数为：

$$
p(x)=\sum_{k=1}^K\alpha_kp(x|\mu_k,\Sigma_k)\tag{2}
$$

其中， $p(x|\mu_k,\Sigma_k)$ 表示参数为 $\mu_k,\Sigma_k$ 的高斯分布的概率密度。

我们称 (2) 式为一个高斯混合(Mixture of Gaussians, Gaussian Mixture)。其中每个高斯密度函数称为混合的一个分模型(component)，有自己的均值 $\mu_k$ 和协方差矩阵 $\Sigma_k$ 。

(2) 式中的参数 $\alpha_k$ 是模型的混合系数 (mixing coefficients)。将(2) 式左右两侧对 $x$ 积分，得到

$$\sum_{k=1}^K\alpha_k=1$$

此外，由于$p(x)\geq0$， $p(x|\mu_k,\Sigma_k)\geq0$，所以 $\alpha_k\geq0$ 。即混合系数应满足 $0\leq\alpha_k\leq1$ 。

(更一般地，混合模型也可以是其他任意分布的叠加。)

由全概率公式， $x$ 的边缘分布 (marginal distribution) 的概率密度为：

$$p(x)=\sum_{k=1}^Kp(k)p(x|k)$$

上式与 (2) 式等价，其中 $p(k)=\alpha_k$ 可以看作选择第 $k$ 个分模型的先验概率 (prior probability)， $p(x|k)$ 是 $x$ 对 $k$ 的条件概率密度。

在观测到 $x$ 后，其来自第 $k$ 个分模型的后验概率 (posterior probability) $p(k|x)$ 称为第 $k$ 个分模型的响应度 (responsibility)。

下图所示为包含两个一维分模型的高斯混合：

![两个一维分模型的高斯混合](https://pic4.zhimg.com/v2-085fa662e29839904957f644bb6a543f_b.jpg)
<label><center>两个一维分模型的高斯混合</center></label>
</br>

###3. 隐变量 & 完全数据
-------------

引入一个 $K$ 维的二值型随机变量 $z=[z_1,\cdots,z_k]$ ，来表示样本 $x$ 由哪一分模型产生。 $z$ 满足这样的条件：$z_k\in\{0,1\}$ ，且 $\sum_{k=1}^Kz_k=1$ ，即其 $K$ 个元素中，有且只有一个为 1，其余为 0。 $z_k=1$ 表示样本 $x$ 由分模型 $k$ 抽样得到。可以看出， $z$ 一共有 $K$ 种可能的状态。

$z$ 的边缘分布由混合系数给出： $p(z_k=1)=\alpha_k$ 。也可写成如下形式：

$$p(z)=\prod_{k=1}^K\alpha_k^{z_k}$$

考虑由以下方式产生样本 $x$：

1.  先以离散分布 $p(z)$ 抽样得到变量 $z$ ；
2.  设根据 $z$ 的取值选择了第 $k$ 个分模型，则以高斯分布 $p(x|\mu_k,\Sigma_k)$  抽样得到 $x$ 。

记高斯混合模型的参数为 $\alpha\equiv\{\alpha_1,\cdots,\alpha_K\}$ ，$\mu\equiv\{\mu_1,\cdots,\mu_K\}$ ， $\Sigma\equiv\{\Sigma_1,\cdots,\Sigma_K\}$ ，则这个过程可由如下的 graphical model 表示 [1]：

![](https://pic1.zhimg.com/v2-1a7383c34ca2e831dd6e490fac24a060_b.jpg)
 <label><center>高斯混合模型的 graphical model</center></label>
<br>

变量 $z$ 称为隐变量 (latent variable)，包含 $(x,z)$ 取值的样本称为完全数据 (complete data)，只含有 $x$ 取值的样本称为不完全数据 (incomplete data)，

给定 $z$ 后 $x$ 的条件概率密度为：

$$p(x|z_k=1)=p(x|\mu_k,\Sigma_k)$$

或者写成如下形式：

$$p(x|z)=\prod_{k=1}^Kp(x|\mu_k,\Sigma_k)^{z_k}$$

$x$ 的边缘分布为联合概率分布 $p(x,z)$ 对 $z$ 的所有可能状态求和：

$$\begin{aligned}
p(x)&=\sum_zp(x,z)=\sum_zp(z)p(x|z)\\
&=\sum_z\bigg(\prod_{k=1}^K\alpha_k^{z_k}\prod_{k=1}^Kp(x|\mu_k,\Sigma_k)^{z_k}\bigg)\\
&=\sum_z\bigg(\prod_{k=1}^K[\alpha_kp(x|\mu_k,\Sigma_k)]^{z_k}\bigg)\\
&=\sum_{k=1}^K\alpha_kp(x|\mu_k,\Sigma_k)
\end{aligned}$$

也可由下面的式子得到：

$$p(x)=\sum_{k=1}^Kp(z_k=1)p(x|z_k=1)=\sum_{k=1}^K\alpha_kp(x|\mu_k,\Sigma_k)$$

这表明 $x$ 的边缘分布就是高斯混合的形式。

###4. 后验概率 & 响应度
-------------

根据贝叶斯定理 ([Bayes' theorem - Wikipedia](https://link.zhihu.com/?target=https%3A//en.wikipedia.org/wiki/Bayes%2527_theorem))，观测到 $x$ 后，其来自第 $k$ 个分模型的后验概率 (posterior responsibility) 为：

$$p(z_k=1|x)=\frac{p(z_k=1)p(x|z_k=1)}{p(x)}=\frac{\alpha_kp(x|\mu_k,\Sigma_k)}{\sum_{j=1}^K\alpha_kp(x|\mu_k,\Sigma_k)}\tag{3}$$

上式中， $p(z_k=1|x)$ 和 $p(z_k=1)$ 为概率， $p(x|z_k=1)$ 和 $p(x)$ 为概率密度。

将上式定义为： $\gamma(z_k)=p(z_k=1|x)$ ，称为第 $k$ 个分模型对 $x$ 的响应度 (responsibility)[2]。

对于样本集 $X=\lbrace x_1,\cdots,x_N\rbrace$ ，记 $x_n$ 对应的隐变量为 $z_n=[z_{n1},\cdots,z_{nK}], n=1,\cdots,N$ ，则第 $k$ 个分模型对 $x_n$ 的响应度为：

$$\gamma(z_{nk})\equiv p(z_{nk}=1|x_n)=\frac{\alpha_kp(x_n|\mu_k,\Sigma_k)}{\sum_{k=1}^K\alpha_kp(x_n|\mu_k,\Sigma_k)}\tag{4}$$

###5. 对数似然函数 & 最大似然估计
------------------

现在有一个样本集 $X=\lbrace x_1,\cdots,x_N\rbrace$ ，我们假设 $X$ 是由一个高斯混合模型产生的，要估计这个模型的参数： $\alpha\equiv\{\alpha_1,\cdots,\alpha_K\}$ ，$\mu\equiv\{\mu_1,\cdots,\mu_K\}$ ， $\Sigma\equiv\{\Sigma_1,\cdots,\Sigma_K\}$ 。

样本集 $X$ (不完全数据)的似然函数 (likelihood function) 为：

$$p(X|\alpha,\mu,\Sigma)=\prod_{n=1}^N\Bigg\lbrack\sum_{k=1}^K\alpha_kp(x_n|\mu_k,\Sigma_k)\Bigg\rbrack$$

似然函数中的连乘求导比较麻烦，取自然对数将其转换成对数的和，得到对数似然函数 (the log of the likelihood function)：

$$\ln p(X|\alpha,\mu,\Sigma)=\sum_{n=1}^N\ln\Bigg\lbrack\sum_{k=1}^K\alpha_kp(x_n|\mu_k,\Sigma_k)\Bigg\rbrack$$

其中，

$$p(x_n|\mu_k,\Sigma_k)=\frac{1}{(2\pi)^{D/2}\Sigma_k^{1/2}}\exp\bigg\lbrace -\frac{1}{2}(x_n-\mu_k)^T\Sigma^{-1}(x_n-\mu_k)\bigg\rbrace$$


最大似然估计 (maximum likelihood estimation)，即通过求似然函数的最大值来估计概率模型的参数。用最大似然估计来计算高斯混合模型的参数，形成如下的优化问题：

$$\begin{aligned}
\max_{\alpha,\mu,\Sigma}\ln p(X|\alpha,\mu,\Sigma)\\
s.t. \sum_{k=1}^K\alpha_k=1
\end{aligned}$$

采用拉格朗日乘子法来求极值，拉格朗日函数为：

$$L(\alpha,\mu,\Sigma)=\ln p(X|\mu,\Sigma)+\lambda\Bigg(\sum_{k=1}^K\alpha_k-1\Bigg)$$

先将 $L(\alpha,\mu,\Sigma)$ 对 $\mu_k$ 求梯度。因为

$$\nabla_{\mu_k}p(x_n|\mu_k,\Sigma_k)=p(x_n|\mu_k,\Sigma_k)[\Sigma_k^{-1}(x_n-\mu_k)]$$

所以，

$$\begin{aligned}
\nabla_{\mu_k}L&=\frac{\partial\ln p(X|\alpha,\mu,\Sigma)}{\partial\mu_k}\\
&=\frac{\partial\big(\sum_{n=1}^N\ln[\sum_{k=1}^K\alpha_kp(x_n|\mu_k,\Sigma_k)]\big)}{\partial\mu_k}\\
&=\sum_{n=1}^N\frac{\partial\ln[\sum_{k=1}^K\alpha_kp(x_n|\mu_k,\Sigma_k)]}{\partial\mu_k}\\
&=\sum_{n=1}^N
\Bigg\lbrack\frac{\alpha_kp(x_n|\mu_k,\Sigma_k)}{\sum_{j=1}^K\alpha_jp(x_n|\mu_j,\Sigma_j)}[\Sigma_k^{-1}(x_n-\mu_k)]\Bigg\rbrack\end{aligned}$$


注意上式中 $\frac{\alpha_kp(x_n|\mu_k,\Sigma_k)}{\sum_{k=1}^K\alpha_kp(x_n|\mu_k,\Sigma_k)}$ 即为响应度 $\gamma(z_{nk})$ ，所以有：

$$\nabla_{\mu_k}L=\sum_{n=1}^N\big[\gamma(z_{nk})[\Sigma_k^{-1}(x_n-\mu_k)]\big]$$

-----------------------------------------------------------------

令 $\nabla_{\mu_k}L=0$ ，则 $\sum_{n=1}^N\big[\gamma(z_{nk})[\Sigma_k^{-1}(x_n-\mu_k)]\big]=0$ ，

左乘 $\Sigma_k$ ，整理得

$$\sum_{n=1}^N[\gamma(z_{nk})x_n]=\sum_{n=1}^N[\gamma(z_nk)\mu_k]=\mu_k\sum_{n=1}^N\gamma(z_{nk})$$

定义 $N_k=\sum_{n=1}^N\gamma(z_{nk})$ ，则

$$\mu_k=\frac{1}{N_k}\sum_{n=1}^N[\gamma(z_{nk})x_n]$$

$N_k$可理解为被分配到第 $k$ 个分模型 (聚类) 的“有效“的样本数。

对$\Sigma_k$中各元素求偏导：

$$\begin{aligned}
\nabla_{\Sigma_k}L&=\frac{\partial{\ln p(X|\alpha,\mu,\Sigma)}}{\partial{\Sigma_k}}\\
&=\frac{\partial\sum_{n=1}^N\ln\big[\sum_{k=1}^K\alpha_kp(x_n|\mu_k,\Sigma_k)\big]}{\partial\Sigma_k}\\
&=\sum_{n=1}^N\frac{\partial\ln\big[\sum_{k=1}^K\alpha_kp(x_n|\mu_k,\Sigma_k)\big]}{\partial\Sigma_k}\\
&=\sum_{n=1}^N\Big[\frac{\alpha_k}{\sum_{j=1}^K\alpha_jp(x_n|\mu_j,\Sigma_j)}\frac{\partial p(x_n|\mu_k,\Sigma_k)}{\partial\Sigma_k}\Big]
\end{aligned}$$

接下来涉及矩阵求导 ([Matrix calculus - Wikipedia](https://link.zhihu.com/?target=https%3A//en.wikipedia.org/wiki/Matrix_calculus))，要复杂一些，这里不做推导，按参考文献 [1]Chapter 9 的公式 (9.19)，给出 $\nabla_{\Sigma_k}L=0$ 的结果：

$$\Sigma_k=\frac{1}{N_k}\sum_{n=1}^N\gamma(z_{nk})(x_n-\mu_k)(x_n-\mu_k)^T$$

对 $\alpha_k$ 求偏导并令其为 0：

$$\begin{aligned}
\nabla_{\alpha_k}L&=\frac{\partial\ln p(X|\alpha,\mu,\Sigma)}{\partial\alpha_k}\\
&=\frac{\partial\Big[\sum_{n=1}^N\ln\big[\sum_{j=1}^K\alpha_jp(x_n|\mu_j,\Sigma_j)\big]+\lambda(\sum_{j=1}^K\alpha_j-1)\Big]}{\partial\alpha_k}\\
&=\sum_{n=1}^N\frac{p(x_n|\mu_k,\Sigma_k)}{\sum_{j=1}^K\alpha_jp(x_n|\mu_j,\Sigma_j)}+\lambda=0
\end{aligned}$$

注意到上式左右两边乘以 $\alpha_k$ 可凑出$\gamma(z_{nk})$ ，所以有

$$\sum_{n=1}^N\gamma(z_{nk})+\alpha_k\lambda=N_k+\alpha_k\lambda=0$$

上式对 $k$ 求和得

$$\sum_{k=1}^K(N_k+\alpha_k\lambda)=\sum_{k=1}^KN_k+\lambda\sum_{k=1}^K\alpha_k=\sum_{k=1}^KN_k+\lambda=0$$

又

$$\begin{aligned}
\sum_{k=1}^KN_k&=\sum_{k=1}^K\sum_{n=1}^N\gamma(z_{nk})=\sum_{n=1}^N\sum_{k=1}^K\gamma(z_{nk})\\
&=\sum_{n=1}^N\sum_{k=1}^K\frac{\alpha_kp(x_n|\mu_k,\Sigma_k)}{\sum_{j=1}^K\alpha_jp(x_n|\mu_j,\Sigma_j)}\\
&=\sum_{n=1}^N1=N
\end{aligned}$$

所以 $\lambda=-N$，进而 $\alpha_k=\frac{N_k}{N}$ 。

综上，对数似然函数 $\ln p(X|\alpha,\mu,\Sigma)$ 的极值点满足的条件为：

$$\begin{cases}
\mu_k=\frac{1}{N_k}\sum_{n=1}^N[\gamma(z_{nk})x_n]\\
\Sigma_k=\frac{1}{N_k}\sum_{n=1}^N\gamma(z_{nk})(x_n-\mu_k)(x_n-\mu_k)^T\tag{6}\\
\alpha_k=\frac{N_k}{N}
\end{cases}$$

需要注意的是，上式并未给出高斯混合模型的解析解 / 闭式解 (analytical soluiton/closed-form solution)，因为响应度 $\gamma(z_{nk})$ 由式 (4) 给出，而参数$\mu_k,\Sigma_k,\alpha_k$未知，故 $\gamma(z_{nk})$、$N_k$ 无法计算。

不过，根据 (6) 式可使用迭代算法来计算模型参数。

###7. 高斯混合模型的 EM 算法
----------------
虽然 (6) 式不是闭式解，但是我们可以根据 (6) 式，通过EM算法迭代来计算参数$\mu_k,\Sigma_k,\alpha_k$ 。

下面简述用 EM 算法来计算高斯混合模型参数的步骤：

1. 初始化：给参数均值向量 $\mu_k$ ，协方差矩阵 $\Sigma_k$ 和混合系数 $\alpha_k$ 赋初值；并计算对数似然函数的初值；

* 注：可以用 K 均值聚类 (K-means clustering) 算法来得到初始参数。

2. **E 步 (Expectation step)**：用当前参数值$\mu_k,\Sigma_k,\alpha_k$  计算后验概率 (响应度) $\gamma(z_{nk})$ ：

$$\gamma(z_{nk})=\frac{\alpha_kp(x_n|\mu_k,\Sigma_k)}{\sum_{j=1}^K\alpha_jp(x_n|\mu_j,\Sigma_j)}$$

3. **M 步 (Maximization step)**：用当前响应度$\gamma(z_{nk})$ ，根据 (6) 式重新估计参数：

$$\begin{cases}
\mu_k^\bold{new}=\frac{1}{N_k}\sum_{n=1}^N[\gamma(z_{nk})x_n]\\
\Sigma_k^\bold{new}=\frac{1}{N_k}\sum_{n=1}^N\gamma(z_{nk})(x_n-\mu_k)(x_n-\mu_k)^T\\
\alpha_k^\bold{new}=\frac{N_k}{N}
\end{cases}$$

其中， $N_k=\sum_{n=1}^N\gamma(z_{nk})$ 。

在 M 步中，先计算 $\mu_k^\bold{new}$ 的值，然后用$\mu_k^\bold{new}$ 来计算新的协方差矩阵 $\Sigma_k^\bold{new}$ 。

4. 用新的参数 $\mu_k^\bold{new},\Sigma_k^\bold{new},\alpha_k^\bold{new}$ 计算对数似然函数：

$$\ln p(X|\alpha,\mu,\Sigma)=\sum_{n=1}^N\ln\Big[\small\sum_{k=1}^K\normalsize \alpha_k\mathcal{N}(x_n|\mu_k,\Sigma_k)\Big]$$

然后检查是否收敛。(当然，也可直接检查参数是否收敛。)

如果收敛，则得到了模型参数；如果还没收敛，则返回第 2 步继续迭代。

参考文献 [1], Section9.2, Page437 指出，E 步 + M 步可以保证对数似然函数的上升。不过需要注意的是，作为一种迭代算法，EM 算法有可能收敛到局部最大值 (local maximum)。


###8. EM 算法的一般形式
-------------

对于含有隐变量的概率模型，记变量和 (离散的) 隐变量分别为$x,z$，模型参数为 $\theta$，(比如高斯混合模型的参数$\theta=\lbrace\alpha,\mu,\Sigma\rbrace$ ， $X=\lbrace x_1,\cdots,x_N\rbrace$ 为变量 $x$ 个观测值的集合，即不完全数据集，$Z=\lbrace z_1,\cdots,z_K\rbrace$为相应的隐变量值的集合。

要计算 $\theta$ 极大化对数似然函数 $\ln p(X|\theta)$  ，EM 算法的思路是：考虑完全数据集 ${X,Z}$ 的对数似然函数 $\ln p(X,Z|\theta)$ 在隐变量的后验分布 $p(Z|X,\theta^{\bold{old}})$ 下的期望，反复迭代极大化这个期望，最终得到模型的参数值。其中 $\theta^{\bold{old}}$ 为当前参数。(若 $x_1,\cdots,x_N$ 相互独立，则计算 $p(Z|X,\theta^{\bold{old}})$ 实际上是计算 $p(z_n|x_n,\theta^{\bold{old}}),n=1,\cdots,N$ )

下面简述 EM 算法的一般形式：

1. 初始化模型参数 $\theta^{\bold{old}}$ ；

2. **E 步 (Expectation step)**：计算隐变量的后验分布 $p(Z|X,\theta^{\bold{old}})$ ；

* 对于高斯混合模型，这一步实际上就是计算响应度 $\gamma(z_{nk})$ 。

3. **M 步 (Maximization step)**：极大化完全数据的对数似然函数 $\ln p(X,Z|\theta)$  在隐变量的后验分布 $p(Z|X,\theta^{\bold{old}})$ 下的期望，即极大化函数:

$$Q(\theta,\theta^{\bold{old}})=\sum_Zp(Z|X,\theta^{\bold{old}})\ln p(X,Z|\theta)$$

得到相应的参数值 $\theta^{\bold{new}}=\arg\max_\theta Q(\theta,\theta^{\bold{old}})$  。

4. 检查对数似然函数或参数是否收敛，如果没收敛，则

$$\theta^{\bold{old}}\larr\theta^{\bold{new}}$$

回到第 2 步继续迭代；

如果已收敛，则得到了模型参数。

EM 算法的每次循环可以使不完全数据集的对数似然函数上升，除非其已收敛到局部最大值 (参考文献 [1], Section9.3, Page440)。

###9. 换个角度看高斯混合模型的 EM 算法
---------------------

在前面的极大似然估计中，我们的目标函数是不完全数据集$X=\lbrace x_1,\cdots,x_N\rbrace$ 的对数似然函数 $\ln p(X|\alpha,\mu,\Sigma)$ 。

现在我们换个角度来看：假如我们也有隐变量的观测值 $Z=\lbrace z_1,\cdots,z_N\rbrace$，即知道 $x_n$ 来自高斯混合中的哪一个分模型，换句话说，我们有完全数据集$\lbrace X,Z \rbrace$ ，那么我们可以计算完全数据的对数似然函数 $\ln p(X,Z|\alpha,\mu,\Sigma)$ 。

首先，一组完全数据 $\lbrace X,Z \rbrace$ 的似然函数为：

$$p(x_n,z_n|\alpha,\mu,\Sigma)=\prod_{k=1}^K\alpha_k^{z_{nk}}p(x_n|\mu_k,\Sigma_k)^{z_{nk}}$$

所以，整个完全数据集 $\lbrace X,Z \rbrace$ 的似然函数为：

$$p(X,Z|\alpha,\mu,\Sigma)=\prod_{n=1}^N\prod_{k=1}^K\alpha_k^{z_{nk}}p(x_n|\mu_k,\Sigma_k)^{z_{nk}}$$

(当然，我们假设每组数据相互独立。)

取对数得

$$\ln p(X,Z|\alpha,\mu,\Sigma)=\sum_{n=1}^N\sum_{k=1}^K\lbrace z_{nk}[\ln\alpha_k+\ln p(x_n|\mu_k,\Sigma_k)]\rbrace\tag{7}$$

可以看出，和不完全数据的对数似然函数 $\small\ln p(X|\alpha,\mu,\Sigma)=\sum_{n=1}^N\ln\Big[\small\sum_{k=1}^K\normalsize \alpha_kp(x_n|\mu_k,\Sigma_k)\Big]$ 相比，(7) 式中 $\sum_{k=1}^K$ 和 $\ln$ 换了顺序。

然而实际情况是，我们并不知道隐变量的观测值，即 (7) 式中的 $z_{nk}$ ，所以我们无法直接计算 $\small\ln p(X,Z|\alpha,\mu,\Sigma)$  。因此，我们考虑对 $\small\ln p(X,Z|\alpha,\mu,\Sigma)$  在隐变量 $z_{nk}$ 的后验分布下求期望，利用求期望的过程，将 $z_{nk}$ 消掉。

$\small\ln p(X,Z|\alpha,\mu,\Sigma)$ 在隐变量 $z_{nk}$ 的后验分布下的期望：

$$\begin{aligned}
E_Z\ln p(X,Z|\alpha,\mu,\Sigma)&=E\sum_{n=1}^N\sum_{k=1}^K\lbrace z_{nk}[\ln\alpha_k+\ln p(x_n|\mu_k,\Sigma_k)]\rbrace\\
&=\sum_{n=1}^N\sum_{k=1}^K\lbrace E(z_{nk})[\ln\alpha_k+\ln p(x_n|\mu_k,\Sigma_k)]\rbrace
\end{aligned}$$

其中， $E(z_{nk})$ 表示 $z_{nk}$ 在后验分布下的期望 $E(z_{nk}|X,\theta)$ 。因 $z_{nk}\in\{0,1\}$ ，所以

$$\begin{aligned}
E(z_{nk}|X,\theta)&=0\times p(z_{nk}=0|X,\theta)+1\times p(z_{nk}=1|X,\theta)\\
&=p(z_{nk}=1|X,\theta)\\
&=\gamma(z_{nk})
\end{aligned}$$

也就是说，期望  $E(z_{nk})$ 就是响应度 $\gamma(z_{nk})$ 。进而，

$$E_Z\ln p(X,Z|\alpha,\mu,\Sigma)=\sum_{n=1}^N\sum_{k=1}^K\lbrace \gamma(z_{nk})[\ln\alpha_k+\ln p(x_n|\mu_k,\Sigma_k)]\rbrace$$

将上式简记为 $E_Z$ ，现在将 $\gamma(z_{nk})$ 当作常数，求参数 $\alpha,\mu,\Sigma$ ，使 $E_Z$ 最大化，即

$$\begin{aligned}
&\max_{\alpha,\mu,\Sigma}E_Z\\
&s.t. \sum_{k=1}^K\alpha_k=1\\
\end{aligned}$$

采用拉格朗日乘子法，拉格朗日函数为：

$$L_E=E_Z+\lambda\Bigg(\small\sum_{k=1}^K\alpha_k-1\normalsize\Bigg)$$

先对$\mu_k$ 求梯度，因为

$$\small \frac{\partial\ln p(x_n|\mu_k,\Sigma_k)}{\partial\mu_k}=\Sigma_k^{-1}(x_n-\mu_k)$$

所以，

$$\begin{aligned}
\nabla_{\mu_k}L_E&=\sum_{n=1}^N\Bigg[\frac{\partial\sum_{k=1}^K\gamma(z_{nk})\ln p(x_n|\mu_k,\Sigma_k)}{\partial\mu_k}\Bigg]\\
&=\sum_{n=1}^N\Bigg[\gamma(z_{nk})\frac{\partial\ln p(x_n|\mu_k,\Sigma_k)}{\partial\mu_k}\Bigg]\\
&=\sum_{n=1}^N\gamma(z_{nk})[\Sigma^{-1}(x_n-\mu_k)]
\end{aligned}$$

进而，

$$\begin{aligned}
\frac{\partial L_E}{\partial\mu_k}=0&\rArr \sum_{n=1}^N\gamma(z_{nk})[\Sigma_k^{-1}(x_n-\mu_k)]=0\\
&\rArr\sum_{n=1}^N\gamma(z_{nk})(x_n-\mu_k)=0\\
&\rArr\sum_{n=1}^N\gamma(z_{nk})x_n=\sum_{n=1}^N\gamma(z_{nk})\mu_k
\end{aligned}$$

同之前极大化 $\ln p(X|\alpha,\mu,\Sigma)$ 一样，可推得

$$\mu_k=\frac{1}{N_k}\sum_{n=1}^N[\gamma(z_{nk})x_n]$$

然后是对$\Sigma_k$ 求梯度，

$$\begin{aligned}
\nabla_{\Sigma_k}L_E&=\sum_{n=1}^N\Bigg[\frac{\partial\sum_{k=1}^K
\gamma(z_{nk})\ln p(x_n|\mu_k,\Sigma_k)}{\partial\Sigma_k}\Bigg]\\
&=\sum_{n=1}^N\Bigg[\gamma(z_{nk})\frac{\partial\ln p(x_n|\mu_k,\Sigma_k)}{\partial\Sigma_k}\Bigg]\\
&=\sum_{n=1}^N\Bigg[\frac{\gamma(z_{nk})}{p(x_n|\mu_k,\Sigma_k)}\frac{\partial p(x_n|\mu_k,\Sigma_k)}{\partial\Sigma_k}\Bigg]\tag{8}\\
\end{aligned}$$

回忆在文 (一) 中，我们求$\ln p(X|\alpha,\mu,\Sigma)$ 的极值点时，有

$$\frac{\partial L}{\partial\Sigma_k}=\sum_{n=1}^N\Big[\frac{\alpha_k}{\sum_{j=1}^K\alpha_jp(x_n|\mu_j,\Sigma_j)}\frac{\partial p(x_n|\mu_k,\Sigma_k)}{\partial\Sigma_k}\Big]\tag{9}$$

对比 (8)(9) 两式，结合响应度 $\gamma(z_{nk})$ 的公式，可发现两式是一致的，这意味着两种方法计算出的 $\Sigma_k$ 也相同。

再来看 $\alpha_k$ ，

$$\begin{aligned}\nabla_{\alpha_k}L_E&=\sum_{n=1}^N\Bigg[\frac{\partial\sum_{k=1}^K\gamma(z_{nk})\ln\alpha_k}{\partial\alpha_k}\Bigg]+\lambda\\
&=\sum_{n=1}^N\Bigg[\gamma(z_{nk})\frac{\partial\ln\alpha_k}{\partial\alpha_k}\Bigg]+\lambda\\
&=\frac{1}{\alpha_k}\sum_{n=1}^N\gamma(z_{nk})+\lambda\\
&=\frac{N_k}{\alpha_k}+\lambda
\end{aligned}$$

对比前文，可以发现 $\alpha_k$ 也和 $\ln p(X|\alpha,\mu,\Sigma)$ 极值点的参数一致。

综上，通过极大化$E_Z(X,Z|\alpha,\mu,\Sigma)$ 得到的模型参数，和 $\ln p(X|\alpha,\mu,\Sigma)$ 的极大似然估计结果是一致的。在 EM 算法的每一次迭代中，我们都在对 $E_Z$ 求极大值。

这给了我们另一个角度来看 EM 算法在高斯混合模型中的应用，也有助于我们理解 EM 算法的一般形式。


参考文献
----

[1] Christopher M. Bishop. Pattern Recognition and Machine Learning, Springer, 2006.

[2] 李航，统计学习方法，清华大学出版社，2012 年。