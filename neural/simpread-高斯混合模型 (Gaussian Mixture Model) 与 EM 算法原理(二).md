> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 https://zhuanlan.zhihu.com/p/61103099

接前文。

6. 高斯混合模型的 EM 算法
----------------

前文讲到，不完全数据集 ![](https://www.zhihu.com/equation?tex=%5Cmathbf%7BX%7D) 的对数似然函数 ![](https://www.zhihu.com/equation?tex=%5Cln+p%28%5Cmathbf%7BX%7C%5Cpi%2C%5Cmu%2C%5CSigma%7D%29+%3D++%5Csum_%7Bn%3D1%7D%5E%7BN%7D%7B%5Cln%5Cleft%5B+%5Csum_%7Bk%3D1%7D%5E%7BK%7D%7B%5Cpi_k+p%5Cleft%28%5Cmathbf%7Bx_n+%7C+%5Cmu_k%2C+%5CSigma_k%7D%5Cright%29%7D+%5Cright%5D%7D) 的极大似然估计得到参数：

![](https://www.zhihu.com/equation?tex=%5Cleft%5C%7B+%5Cbegin%7Balign%7D+%5Cmathbf%7B%5Cmu_k%7D+%26%3D+%5Cfrac%7B1%7D%7BN_k%7D+%5Csum_%7Bn%3D1%7D%5E%7BN%7D%7B%5Cleft%5B+%5Cgamma%28z_%7Bnk%7D%29%5Cmathbf%7Bx_n%7D+%5Cright%5D%7D%5C%5C+%5Cmathbf%7B%5CSigma_k%7D+%26%3D+%5Cfrac%7B1%7D%7BN_k%7D+%5Csum_%7Bn%3D1%7D%5E%7BN%7D%7B%5Cgamma%28z_%7Bnk%7D%29%28%5Cmathbf%7Bx_n-%5Cmu_k%7D%29%28%5Cmathbf%7Bx_n-%5Cmu_k%7D%29%5ET%7D+%5C%5C+%5Cpi_k+%26%3D+%5Cfrac%7BN_k%7D%7BN%7D+%5Cend%7Balign%7D%5C%5C+%5Cright.+%5Ctag%7B1%7D)

其中，响应度 ![](https://www.zhihu.com/equation?tex=%5Cgamma%28z_%7Bnk%7D%29+%3D+%5Cfrac%7B%5Cpi_k+p%5Cleft%28%5Cmathbf%7Bx_n+%7C+%5Cmu_k%2C+%5CSigma_k%7D%5Cright%29%7D+%7B%5Csum_%7Bj%3D1%7D%5E%7BK%7D%7B%5Cpi_j+p%5Cleft%28%5Cmathbf%7Bx_n+%7C+%5Cmu_j%2C+%5CSigma_j%7D%5Cright%29%7D+%7D) ， ![](https://www.zhihu.com/equation?tex=+N_k+%3D+%5Csum_%7Bn%3D1%7D%5E%7BN%7D%7B%5Cgamma%28z_%7Bnk%7D%29%7D) 。

虽然 (1) 式不是闭式解，但是我们可以根据 (1) 式，通过迭代的方法来计算参数 ![](https://www.zhihu.com/equation?tex=%5Cmathbf%7B%5Cmu_k%2C+%5CSigma_k%7D%2C+%5Cpi_k) 。

**EM 算法 (Expectation-Maximization algorithm)** 是就是这样一种用于计算含有隐变量的模型的极大似然解的强大方法。

下面简述用 EM 算法来计算高斯混合模型参数的步骤：

1. 初始化：给参数均值向量 ![](https://www.zhihu.com/equation?tex=%5Cmathbf%7B%5Cmu_k%7D) ，协方差矩阵 ![](https://www.zhihu.com/equation?tex=%5Cmathbf%7B%5CSigma_k%7D) 和混合系数 ![](https://www.zhihu.com/equation?tex=%5Cpi_k) 赋初值；并计算对数似然函数的初值；

* 注：可以用 K 均值聚类 (K-means clustering) 算法来得到初始参数。

2. **E 步 (Expectation step)**：用当前参数值 ![](https://www.zhihu.com/equation?tex=%5Cmathbf%7B%5Cmu_k%2C+%5CSigma_k%7D%2C+%5Cpi_k) 计算后验概率 (响应度) ![](https://www.zhihu.com/equation?tex=%5Cgamma%28z_%7Bnk%7D%29) ：

![](https://www.zhihu.com/equation?tex=%5Cgamma%28z_%7Bnk%7D%29+%5Cleftarrow+%5Cfrac%7B%5Cpi_k+p%5Cleft%28%5Cmathbf%7Bx_n+%7C+%5Cmu_k%2C+%5CSigma_k%7D%5Cright%29%7D+%7B%5Csum_%7Bj%3D1%7D%5E%7BK%7D%7B%5Cpi_j+p%5Cleft%28%5Cmathbf%7Bx_n+%7C+%5Cmu_j%2C+%5CSigma_j%7D%5Cright%29%7D+%7D+%5C%5C)

3. **M 步 (Maximization step)**：用当前响应度 ![](https://www.zhihu.com/equation?tex=%5Cgamma%28z_%7Bnk%7D%29) ，根据 (1) 式重新估计参数：

![](https://www.zhihu.com/equation?tex=%5Cbegin%7Balign%7D++%5Cmathbf%7B%5Cmu_k%5E%7Bnew%7D%7D+%26%5Cleftarrow+%5Cfrac%7B1%7D%7BN_k%7D+%5Csum_%7Bn%3D1%7D%5E%7BN%7D%7B%5Cleft%5B+%5Cgamma%28z_%7Bnk%7D%29%5Cmathbf%7Bx_n%7D+%5Cright%5D%7D%5C%5C++%5Cmathbf%7B%5CSigma_k%5E%7Bnew%7D%7D+%26%5Cleftarrow+%5Cfrac%7B1%7D%7BN_k%7D+%5Csum_%7Bn%3D1%7D%5E%7BN%7D%7B%5Cgamma%28z_%7Bnk%7D%29%28%5Cmathbf%7Bx_n-%5Cmu_k%5E%7Bnew%7D%7D%29%28%5Cmathbf%7Bx_n-%5Cmu_k%5E%7Bnew%7D%7D%29%5ET%7D+%5C%5C++%5Cpi_k%5E%7Bnew%7D+%26%5Cleftarrow+%5Cfrac%7BN_k%7D%7BN%7D+%5Cend%7Balign%7D%5C%5C)

其中， ![](https://www.zhihu.com/equation?tex=+N_k+%3D+%5Csum_%7Bn%3D1%7D%5E%7BN%7D%7B%5Cgamma%28z_%7Bnk%7D%29%7D) 。

在 M 步中，先计算 ![](https://www.zhihu.com/equation?tex=%5Cmathbf%7B%5Cmu_k%5E%7Bnew%7D%7D) 的值，然后用 ![](https://www.zhihu.com/equation?tex=%5Cmathbf%7B%5Cmu_k%5E%7Bnew%7D%7D) 来计算新的协方差矩阵 ![](https://www.zhihu.com/equation?tex=%5Cmathbf%7B%5CSigma_k%5E%7Bnew%7D%7D) 。

4. 用新的参数 ![](https://www.zhihu.com/equation?tex=%5Cmathbf%7B%5Cmu_k%5E%7Bnew%7D%7D%2C+%5Cmathbf%7B%5CSigma_k%5E%7Bnew%7D%7D%2C+%5Cpi_k%5E%7Bnew%7D+) 计算对数似然函数：

![](https://www.zhihu.com/equation?tex=%5Cln+p%28%5Cmathbf%7BX%7C%5Cpi%2C%5Cmu%2C%5CSigma%7D%29+%3D++%5Csum_%7Bn%3D1%7D%5E%7BN%7D%7B%5Cln%5Cleft%5B+%5Csum_%7Bk%3D1%7D%5E%7BK%7D%7B%5Cpi_k+p%5Cleft%28%5Cmathbf%7Bx_n+%7C+%5Cmu_k%2C+%5CSigma_k%7D%5Cright%29%7D+%5Cright%5D%7D%5C%5C)

然后检查是否收敛。(当然，也可直接检查参数是否收敛。)

如果收敛，则得到了模型参数；如果还没收敛，则返回第 2 步继续迭代。

参考文献 [1], Section9.2, Page437 指出，E 步 + M 步可以保证对数似然函数的上升。不过需要注意的是，作为一种迭代算法，EM 算法有可能收敛到局部最大值 (local maximum)。

7. 换个角度看高斯混合模型的 EM 算法
---------------------

在前面的极大似然估计中，我们的目标函数是不完全数据集 ![](https://www.zhihu.com/equation?tex=%5Cmathbf%7BX%3D%5Cleft%5C%7B+x_1%2C+%5Cldots%2C+x_N+%5Cright%5C%7D%7D) 的对数似然函数 ![](https://www.zhihu.com/equation?tex=%5Cln+p%28%5Cmathbf%7BX%7C%5Cpi%2C%5Cmu%2C%5CSigma%7D%29) 。

现在我们换个角度来看：假如我们也有隐变量的观测值 ![](https://www.zhihu.com/equation?tex=%5Cmathbf%7BZ%3D%5Cleft%5C%7B+z_1%2C+%5Cldots%2C+z_N+%5Cright%5C%7D%7D) ，即知道 ![](https://www.zhihu.com/equation?tex=%5Cmathbf%7Bx_n%7D) 来自高斯混合中的哪一个分模型，换句话说，我们有完全数据集 ![](https://www.zhihu.com/equation?tex=%5Cleft%5C%7B+%5Cmathbf%7BX%2C+Z%7D+%5Cright%5C%7D) ，那么我们可以计算完全数据的对数似然函数 ![](https://www.zhihu.com/equation?tex=%5Cln+p%28%5Cmathbf%7BX%2C+Z%7C%5Cpi%2C%5Cmu%2C%5CSigma%7D%29) 。

首先，一组完全数据 ![](https://www.zhihu.com/equation?tex=%5Cleft%5C%7B+%5Cmathbf%7Bx_n%2C+z_n%7D+%5Cright%5C%7D) 的似然函数为：

![](https://www.zhihu.com/equation?tex=p%28%5Cmathbf%7Bx_n%2C+z_n%7C%5Cpi%2C%5Cmu%2C%5CSigma%7D%29+%3D++%5Cprod_%7Bk%3D1%7D%5E%7BK%7D%7B%5Cpi_k%5E%7Bz_%7Bnk%7D%7D+p%5Cleft%28%5Cmathbf%7Bx_n+%7C+%5Cmu_k%2C+%5CSigma_k%7D%5Cright%29%5E%7Bz_%7Bnk%7D%7D%7D+%5C%5C)

所以，整个完全数据集 ![](https://www.zhihu.com/equation?tex=%5Cleft%5C%7B+%5Cmathbf%7BX%2C+Z%7D+%5Cright%5C%7D) 的似然函数为：

![](https://www.zhihu.com/equation?tex=p%28%5Cmathbf%7BX%2C+Z%7C%5Cpi%2C%5Cmu%2C%5CSigma%7D%29+%3D++%5Cprod_%7Bn%3D1%7D%5E%7BN%7D%5Cprod_%7Bk%3D1%7D%5E%7BK%7D%7B%5Cpi_k%5E%7Bz_%7Bnk%7D%7D+p%5Cleft%28%5Cmathbf%7Bx_n+%7C+%5Cmu_k%2C+%5CSigma_k%7D%5Cright%29%5E%7Bz_%7Bnk%7D%7D%7D+%5C%5C)

(当然，我们假设每组数据相互独立。)

取对数得

![](https://www.zhihu.com/equation?tex=%5Cln+p%28%5Cmathbf%7BX%2C+Z%7C%5Cpi%2C%5Cmu%2C%5CSigma%7D%29+%3D++%5Csum_%7Bn%3D1%7D%5E%7BN%7D%7B+%5Csum_%7Bk%3D1%7D%5E%7BK%7D%7B%5Cleft%5C%7B+%7Bz_%7Bnk%7D%5Cleft%5B%5Cln+%5Cpi_k+%2B+%5Cln+p%5Cleft%28%5Cmathbf%7Bx_n+%7C+%5Cmu_k%2C+%5CSigma_k%7D%5Cright%29%5Cright%5D%7D%5Cright%5C%7D%7D+%7D%5Ctag%7B2%7D%5C%5C)

可以看出，和不完全数据的对数似然函数 ![](https://www.zhihu.com/equation?tex=%5Cln+p%28%5Cmathbf%7BX%7C%5Cpi%2C%5Cmu%2C%5CSigma%7D%29+%3D++%5Csum_%7Bn%3D1%7D%5E%7BN%7D%7B%5Cln%5Cleft%5B+%5Csum_%7Bk%3D1%7D%5E%7BK%7D%7B%5Cpi_k+p%5Cleft%28%5Cmathbf%7Bx_n+%7C+%5Cmu_k%2C+%5CSigma_k%7D%5Cright%29%7D+%5Cright%5D%7D)相比，(2) 式中 ![](https://www.zhihu.com/equation?tex=%5Csum_%7Bk%3D1%7D%5E%7BK%7D) 和 ![](https://www.zhihu.com/equation?tex=%5Cln) 换了顺序。

然而实际情况是，我们并不知道隐变量的观测值，即 (2) 式中的 ![](https://www.zhihu.com/equation?tex=z_%7Bnk%7D) ，所以我们无法直接计算 ![](https://www.zhihu.com/equation?tex=%5Cln+p%28%5Cmathbf%7BX%2C+Z%7C%5Cpi%2C%5Cmu%2C%5CSigma%7D%29) 。因此，我们考虑对 ![](https://www.zhihu.com/equation?tex=%5Cln+p%28%5Cmathbf%7BX%2C+Z%7C%5Cpi%2C%5Cmu%2C%5CSigma%7D%29) 在隐变量 ![](https://www.zhihu.com/equation?tex=z_%7Bnk%7D) 的后验分布下求期望，利用求期望的过程，将 ![](https://www.zhihu.com/equation?tex=z_%7Bnk%7D) 消掉。

![](https://www.zhihu.com/equation?tex=%5Cln+p%28%5Cmathbf%7BX%2C+Z%7C%5Cpi%2C%5Cmu%2C%5CSigma%7D%29) 在隐变量 ![](https://www.zhihu.com/equation?tex=z_%7Bnk%7D) 的后验分布下的期望：

![](https://www.zhihu.com/equation?tex=%5Cbegin%7Balign%7D+E_%7B%5Cmathbf+Z%7D+%5Cln+p%28%5Cmathbf%7BX%2C+Z%7C%5Cpi%2C%5Cmu%2C%5CSigma%7D%29++%26%3D+E+%5Csum_%7Bn%3D1%7D%5E%7BN%7D%7B+%5Csum_%7Bk%3D1%7D%5E%7BK%7D%7B%5Cleft%5C%7B+%7Bz_%7Bnk%7D%5Cleft%5B%5Cln+%5Cpi_k+%2B+%5Cln+p%5Cleft%28%5Cmathbf%7Bx_n+%7C+%5Cmu_k%2C+%5CSigma_k%7D%5Cright%29%5Cright%5D%7D%5Cright%5C%7D%7D%7D%5C%5C+%26%3D%5Csum_%7Bn%3D1%7D%5E%7BN%7D%7B+%5Csum_%7Bk%3D1%7D%5E%7BK%7D%7B%5Cleft%5C%7BE%28z_%7Bnk%7D%29%5Cleft%5B%5Cln+%5Cpi_k+%2B+%5Cln+p%5Cleft%28%5Cmathbf%7Bx_n+%7C+%5Cmu_k%2C+%5CSigma_k%7D%5Cright%29%5Cright%5D%5Cright%5C%7D%7D%7D+%5Cend%7Balign%7D+%5C%5C)

其中， ![](https://www.zhihu.com/equation?tex=E%28z_%7Bnk%7D%29) 表示 ![](https://www.zhihu.com/equation?tex=z_%7Bnk%7D) 在后验分布下的期望 ![](https://www.zhihu.com/equation?tex=E%28z_%7Bnk%7D%7C%5Cmathbf%7BX%2C+%5Ctheta%7D%29) 。因 ![](https://www.zhihu.com/equation?tex=z_%7Bnk%7D%5Cin+%5C%7B0%2C+1%5C%7D) ，所以

![](https://www.zhihu.com/equation?tex=%5Cbegin%7Balign%7D+E%28z_%7Bnk%7D%7C%5Cmathbf%7BX%2C+%5Ctheta%7D%29+%26%3D+0%5Ctimes+p%28z_%7Bnk%7D%3D0%7C%5Cmathbf%7BX%2C+%5Ctheta%7D%29+%2B+1%5Ctimes+p%28z_%7Bnk%7D%3D1%7C%5Cmathbf%7BX%2C+%5Ctheta%7D%29%5C%5C+%26%3D+p%28z_%7Bnk%7D%3D1%7C%5Cmathbf%7BX%2C+%5Ctheta%7D%29%5C%5C+%26%3D+%5Cgamma%28z_%7Bnk%7D%29+%5Cend%7Balign%7D%5C%5C)

也就是说，期望 ![](https://www.zhihu.com/equation?tex=E%28z_%7Bnk%7D%29) 就是响应度 ![](https://www.zhihu.com/equation?tex=%5Cgamma%28z_%7Bnk%7D%29) 。进而，

![](https://www.zhihu.com/equation?tex=%5Cbegin%7Balign%7D+E_%7B%5Cmathbf+Z%7D%5Cln+p%28%5Cmathbf%7BX%2C+Z%7C%5Cpi%2C%5Cmu%2C%5CSigma%7D%29++%26%3D%5Csum_%7Bn%3D1%7D%5E%7BN%7D%7B+%5Csum_%7Bk%3D1%7D%5E%7BK%7D%7B%5Cleft%5C%7B%5Cgamma%28z_%7Bnk%7D%29%5Cleft%5B%5Cln+%5Cpi_k+%2B+%5Cln+p%5Cleft%28%5Cmathbf%7Bx_n+%7C+%5Cmu_k%2C+%5CSigma_k%7D%5Cright%29%5Cright%5D%5Cright%5C%7D%7D%7D+%5Cend%7Balign%7D+%5C%5C)

将上式简记为 ![](https://www.zhihu.com/equation?tex=E_%7B%5Cmathbf%7BZ%7D%7D) ，现在将 ![](https://www.zhihu.com/equation?tex=%5Cgamma%28z_%7Bnk%7D%29) 当作常数，求参数 ![](https://www.zhihu.com/equation?tex=%5Cmathbf%7B%5Cpi%2C%5Cmu%2C%5CSigma%7D) ，使 ![](https://www.zhihu.com/equation?tex=E_%7B%5Cmathbf%7BZ%7D%7D) 最大化，即

![](https://www.zhihu.com/equation?tex=%5Cmax_%7B%5Cmathbf%7B%5Cpi%2C%5Cmu%2C%5CSigma%7D%7D%7BE_%7B%5Cmathbf%7BZ%7D%7D%7D%5C%5C+s.t.+%5Csum_%7Bk%3D1%7D%5E%7BK%7D%7B%5Cpi_k%7D+%3D+1)

采用拉格朗日乘子法，拉格朗日函数为：

![](https://www.zhihu.com/equation?tex=L_E+%3D+E+%2B+%5Clambda+%5Cleft%28+%5Csum_%7Bk%3D1%7D%5E%7BK%7D%7B%5Cpi_k%7D+-+1+%5Cright%29+%5C%5C)

先对 ![](https://www.zhihu.com/equation?tex=%5Cmathbf%7B%5Cmu_k%7D) 求梯度，因为

![](https://www.zhihu.com/equation?tex=%5Cbegin%7Balign%7D+%5Cfrac%7B%5Cpartial+%5Cln+p%5Cleft%28%5Cmathbf%7Bx_n+%7C+%5Cmu_k%2C+%5CSigma_k%7D%5Cright%29%7D%7B%5Cpartial+%5Cmathbf%7B%5Cmu_k%7D%7D+%26%3D+%5Cfrac%7B1%7D%7Bp%5Cleft%28%5Cmathbf%7Bx_n+%7C+%5Cmu_k%2C+%5CSigma_k%7D%5Cright%29%7D+p%5Cleft%28%5Cmathbf%7Bx_n+%7C+%5Cmu_k%2C+%5CSigma_k%7D%5Cright%29%5Cleft%5B+%5Cmathbf%7B%5CSigma%5E%7B-1%7D%28x_n-%5Cmu_k%29%7D+%5Cright%5D+%5C%5C%26%3D+%5Cmathbf%7B%5CSigma%5E%7B-1%7D%28x_n-%5Cmu_k%29%7D+%5Cend%7Balign%7D+%5C%5C)

所以，

![](https://www.zhihu.com/equation?tex=%5Cbegin%7Balign%7D+%5Cfrac%7B%5Cpartial+L_E%7D%7B%5Cpartial+%5Cmathbf%7B%5Cmu_k%7D%7D+%26%3D+%5Csum_%7Bn%3D1%7D%5E%7BN%7D%7B+%5Cleft%5B+%5Cfrac%7B%5Cpartial+%5Csum_%7Bk%3D1%7D%5E%7BK%7D%7B%5Cgamma%28z_%7Bnk%7D%29%5Cln+p%5Cleft%28%5Cmathbf%7Bx_n+%7C+%5Cmu_k%2C+%5CSigma_k%7D%5Cright%29%7D%7D%7B%5Cpartial+%5Cmathbf%7B%5Cmu_k%7D%7D%5Cright%5D%7D%5C%5C+%26%3D+%5Csum_%7Bn%3D1%7D%5E%7BN%7D%7B+%5Cleft%5B%5Cgamma%28z_%7Bnk%7D%29+%5Cfrac%7B%5Cpartial+%5Cln+p%5Cleft%28%5Cmathbf%7Bx_n+%7C+%5Cmu_k%2C+%5CSigma_k%7D%5Cright%29%7D%7B%5Cpartial+%5Cmathbf%7B%5Cmu_k%7D%7D%5Cright%5D%7D%5C%5C+%26%3D+%5Csum_%7Bn%3D1%7D%5E%7BN%7D%7B%5Cgamma%28z_%7Bnk%7D%29%5Cleft%5B+%5Cmathbf%7B%5CSigma_k%5E%7B-1%7D%28x_n-%5Cmu_k%29%7D+%5Cright%5D%7D%5C%5C+%5Cend%7Balign%7D+%5C%5C)

进而，

![](https://www.zhihu.com/equation?tex=%5Cbegin%7Balign%7D+%5Cfrac%7B%5Cpartial+L_E%7D%7B%5Cpartial+%5Cmathbf%7B%5Cmu_k%7D%7D+%3D+%5Cmathbf%7B0%7D+%26%5CRightarrow+%5Csum_%7Bn%3D1%7D%5E%7BN%7D%7B%5Cgamma%28z_%7Bnk%7D%29%5Cleft%5B+%5Cmathbf%7B%5CSigma_k%5E%7B-1%7D%28x_n-%5Cmu_k%29%7D+%5Cright%5D%7D+%3D+%5Cmathbf%7B0%7D%5C%5C+%26%5CRightarrow+%5Csum_%7Bn%3D1%7D%5E%7BN%7D%7B%5Cgamma%28z_%7Bnk%7D%29%5Cmathbf%7B%28x_n-%5Cmu_k%29%7D%7D+%3D+%5Cmathbf%7B0%7D%5C%5C+%26%5CRightarrow+%5Csum_%7Bn%3D1%7D%5E%7BN%7D%7B%5Cgamma%28z_%7Bnk%7D%29%5Cmathbf%7Bx_n%7D%7D+%3D+%5Csum_%7Bn%3D1%7D%5E%7BN%7D%7B%5Cgamma%28z_%7Bnk%7D%29%5Cmathbf%7B%5Cmu_k%7D%7D%5C%5C+%5Cend%7Balign%7D%5C%5C)

同之前极大化 ![](https://www.zhihu.com/equation?tex=%5Cln+p%28%5Cmathbf%7BX%7C%5Cpi%2C%5Cmu%2C%5CSigma%7D%29) 一样，可推得

![](https://www.zhihu.com/equation?tex=%5Cmathbf%7B%5Cmu_k%7D+%3D+%5Cfrac%7B1%7D%7BN_k%7D+%5Csum_%7Bn%3D1%7D%5E%7BN%7D%7B%5Cleft%5B+%5Cgamma%28z_%7Bnk%7D%29%5Cmathbf%7Bx_n%7D+%5Cright%5D%7D+%5C%5C)

然后是对 ![](https://www.zhihu.com/equation?tex=%5Cmathbf%7B%5CSigma_k%7D) 求梯度，

![](https://www.zhihu.com/equation?tex=%5Cbegin%7Balign%7D+%5Cfrac%7B%5Cpartial+L_E%7D%7B%5Cpartial+%5Cmathbf%7B%5CSigma_k%7D%7D+%26%3D+%5Csum_%7Bn%3D1%7D%5E%7BN%7D%7B+%5Cleft%5B+%5Cfrac%7B%5Cpartial+%5Csum_%7Bk%3D1%7D%5E%7BK%7D%7B%5Cgamma%28z_%7Bnk%7D%29%5Cln+p%5Cleft%28%5Cmathbf%7Bx_n+%7C+%5Cmu_k%2C+%5CSigma_k%7D%5Cright%29%7D%7D%7B%5Cpartial+%5Cmathbf%7B%5CSigma_k%7D%7D%5Cright%5D%7D%5C%5C+%26%3D+%5Csum_%7Bn%3D1%7D%5E%7BN%7D%7B+%5Cleft%5B%5Cgamma%28z_%7Bnk%7D%29+%5Cfrac%7B%5Cpartial+%5Cln+p%5Cleft%28%5Cmathbf%7Bx_n+%7C+%5Cmu_k%2C+%5CSigma_k%7D%5Cright%29%7D%7B%5Cpartial+%5Cmathbf%7B%5CSigma_k%7D%7D%5Cright%5D%7D%5C%5C+%26%3D+%5Csum_%7Bn%3D1%7D%5E%7BN%7D%7B+%5Cleft%5B%5Cfrac%7B%5Cgamma%28z_%7Bnk%7D%29%7D%7Bp%5Cleft%28%5Cmathbf%7Bx_n+%7C+%5Cmu_k%2C+%5CSigma_k%7D%5Cright%29%7D+%5Cfrac%7B%5Cpartial+p%5Cleft%28%5Cmathbf%7Bx_n+%7C+%5Cmu_k%2C+%5CSigma_k%7D%5Cright%29%7D%7B%5Cpartial+%5Cmathbf%7B%5CSigma_k%7D%7D%5Cright%5D%7D%5C%5C+%5Cend%7Balign%7D+%5Ctag%7B3%7D%5C%5C)

回忆在文 (一) 中，我们求 ![](https://www.zhihu.com/equation?tex=%5Cln+p%28%5Cmathbf%7BX%7C%5Cpi%2C%5Cmu%2C%5CSigma%7D%29) 的极值点时，有

![](https://www.zhihu.com/equation?tex=%5Cbegin%7Balign%7D+++%5Cfrac%7B%5Cpartial+L%7D%7B%5Cpartial+%5Cmathbf%7B%5CSigma_k%7D%7D+%3D+%5Csum_%7Bn%3D1%7D%5E%7BN%7D%7B%5Cleft%5B+%5Cfrac%7B%5Cpi_k+%7D%7B%5Csum_%7Bj%3D1%7D%5E%7BK%7D%7B%5Cpi_j+p%5Cleft%28%5Cmathbf%7Bx_n+%7C+%5Cmu_j%2C+%5CSigma_j%7D%5Cright%29%7D%7D+%5Cfrac%7B%5Cpartial+p%5Cleft%28%5Cmathbf%7Bx_n+%7C+%5Cmu_k%2C+%5CSigma_k%7D%5Cright%29%7D%7B%5Cpartial+%5Cmathbf%7B%5CSigma_k%7D%7D+%5Cright%5D%7D++%5Cend%7Balign%7D+%5Ctag%7B4%7D%5C%5C)

对比 (3)(4) 两式，结合响应度 ![](https://www.zhihu.com/equation?tex=%5Cgamma%28z_%7Bnk%7D%29) 的公式，可发现两式是一致的，这意味着两种方法计算出的 ![](https://www.zhihu.com/equation?tex=%5Cmathbf%7B%5CSigma_k%7D) 也相同。

再来看 ![](https://www.zhihu.com/equation?tex=%5Cpi_k) ，

![](https://www.zhihu.com/equation?tex=%5Cbegin%7Balign%7D+%5Cfrac%7B%5Cpartial+L_E%7D%7B%5Cpartial+%5Cpi_k%7D+%26%3D+%5Csum_%7Bn%3D1%7D%5E%7BN%7D%7B+%5Cleft%5B+%5Cfrac%7B%5Cpartial+%5Csum_%7Bk%3D1%7D%5E%7BK%7D%7B%5Cgamma%28z_%7Bnk%7D%29%5Cln+%5Cpi_k%7D%7D%7B%5Cpartial+%5Cpi_k%7D%5Cright%5D%7D+%2B+%5Clambda%5C%5C+%26%3D+%5Csum_%7Bn%3D1%7D%5E%7BN%7D%7B+%5Cleft%5B%5Cgamma%28z_%7Bnk%7D%29+%5Cfrac%7B%5Cpartial+%5Cln+%5Cpi_k%7D%7B%5Cpartial+%5Cpi_k%7D%5Cright%5D%7D+%2B+%5Clambda%5C%5C+%26%3D+%5Csum_%7Bn%3D1%7D%5E%7BN%7D%7B+%5Cleft%5B%5Cgamma%28z_%7Bnk%7D%29+%5Cfrac%7B1%7D%7B%5Cpi_k%7D%5Cright%5D%7D+%2B+%5Clambda+%3D++%5Cfrac%7B1%7D%7B%5Cpi_k%7D%5Csum_%7Bn%3D1%7D%5E%7BN%7D%7B%5Cgamma%28z_%7Bnk%7D%29%7D+%2B+%5Clambda%5C%5C+%26%3D+%5Cfrac%7B1%7D%7B%5Cpi_k%7D+N_k+%2B+%5Clambda+%5Cend%7Balign%7D+%5C%5C)

对比前文，可以发现 ![](https://www.zhihu.com/equation?tex=%5Cpi_k) 也和 ![](https://www.zhihu.com/equation?tex=%5Cln+p%28%5Cmathbf%7BX%7C%5Cpi%2C%5Cmu%2C%5CSigma%7D%29) 极值点的参数一致。

综上，通过极大化 ![](https://www.zhihu.com/equation?tex=E_%7B%5Cmathbf+Z%7D%5Cln+p%28%5Cmathbf%7BX%2C+Z%7C%5Cpi%2C%5Cmu%2C%5CSigma%7D%29++) 得到的模型参数，和 ![](https://www.zhihu.com/equation?tex=%5Cln+p%28%5Cmathbf%7BX%7C%5Cpi%2C%5Cmu%2C%5CSigma%7D%29) 的极大似然估计结果是一致的。在 EM 算法的每一次迭代中，我们都在对 ![](https://www.zhihu.com/equation?tex=E_%7B%5Cmathbf%7BZ%7D%7D) 求极大值。

这给了我们另一个角度来看 EM 算法在高斯混合模型中的应用，也有助于我们理解 EM 算法的一般形式。

8. EM 算法的一般形式
-------------

对于含有隐变量的概率模型，记变量和 (离散的) 隐变量分别为 ![](https://www.zhihu.com/equation?tex=%5Cmathbf%7Bx%2Cz%7D) ，模型参数为 ![](https://www.zhihu.com/equation?tex=%5Cmathbf%7B%5Ctheta%7D) ，(比如高斯混合模型的参数 ![](https://www.zhihu.com/equation?tex=%5Cmathbf%7B%5Ctheta+%3D+%5C%7B+%5Cpi%2C+%5Cmu%2C+%5CSigma+%5C%7D+%7D) )， ![](https://www.zhihu.com/equation?tex=%5Cmathbf%7BX+%3D+%5C%7Bx_1%2C%5Cldots%2C+x_N%5C%7D%7D) 为变量 ![](https://www.zhihu.com/equation?tex=%5Cmathbf%7Bx%7D) 的 ![](https://www.zhihu.com/equation?tex=N) 个观测值的集合，即不完全数据集， ![](https://www.zhihu.com/equation?tex=%5Cmathbf%7BZ+%3D+%5C%7Bz_1%2C%5Cldots%2C+z_N%5C%7D%7D) 为相应的隐变量值的集合。

要计算 ![](https://www.zhihu.com/equation?tex=%5Cmathbf%7B%5Ctheta%7D) 极大化对数似然函数 ![](https://www.zhihu.com/equation?tex=%5Cln+p%28%5Cmathbf%7BX%7C%5Ctheta%7D%29) ，EM 算法的思路是：考虑完全数据集 ![](https://www.zhihu.com/equation?tex=%5Cmathbf%7B%5C%7BX%2CZ%5C%7D%7D) 的对数似然函数 ![](https://www.zhihu.com/equation?tex=%5Cln+p%28%5Cmathbf%7BX%2C+Z%7C%5Ctheta%7D%29) 在隐变量的后验分布 ![](https://www.zhihu.com/equation?tex=p%28%5Cmathbf%7BZ+%7C+X%2C+%5Ctheta%5E%7Bold%7D%7D%29) 下的期望，反复迭代极大化这个期望，最终得到模型的参数值。其中 ![](https://www.zhihu.com/equation?tex=%5Cmathbf%7B%5Ctheta%5E%7Bold%7D%7D) 为当前参数。(若 ![](https://www.zhihu.com/equation?tex=%5Cmathbf%7B+x_1%2C%5Cldots%2C+x_N%7D) 相互独立，则计算 ![](https://www.zhihu.com/equation?tex=p%28%5Cmathbf%7BZ+%7C+X%2C+%5Ctheta%5E%7Bold%7D%7D%29) 实际上是计算 ![](https://www.zhihu.com/equation?tex=p%28%5Cmathbf%7Bz_n+%7C+x_n%2C+%5Ctheta%5E%7Bold%7D%7D%29%2C+n+%3D+1%2C%5Cldots%2C+N) )

下面简述 EM 算法的一般形式：

1. 初始化模型参数 ![](https://www.zhihu.com/equation?tex=%5Cmathbf%7B%5Ctheta%5E%7Bold%7D%7D) ；

2. **E 步 (Expectation step)**：计算隐变量的后验分布 ![](https://www.zhihu.com/equation?tex=p%28%5Cmathbf%7BZ+%7C+X%2C+%5Ctheta%5E%7Bold%7D%7D%29) ；

* 对于高斯混合模型，这一步实际上就是计算响应度 ![](https://www.zhihu.com/equation?tex=%5Cgamma%28z_%7Bnk%7D%29) 。

3. **M 步 (Maximization step)**：极大化完全数据的对数似然函数 ![](https://www.zhihu.com/equation?tex=%5Cln+p%28%5Cmathbf%7BX%2C+Z%7C%5Ctheta%7D%29) 在隐变量的后验分布 ![](https://www.zhihu.com/equation?tex=p%28%5Cmathbf%7BZ+%7C+X%2C+%5Ctheta%5E%7Bold%7D%7D%29) 下的期望，即极大化函数:

![](https://www.zhihu.com/equation?tex=Q%5Cleft%28+%5Cmathbf%7B%5Ctheta%7D%2C+%5Cmathbf%7B%5Ctheta%5E%7Bold%7D%7D+%5Cright%29+%3D+%5Csum_%7B%5Cmathbf%7BZ%7D%7D%7Bp%28%5Cmathbf%7BZ+%7C+X%2C+%5Ctheta%5E%7Bold%7D%7D%29+%5Cln+p%28%5Cmathbf%7BX%2CZ%7C%5Ctheta%7D%29%7D%5C%5C)

得到相应的参数值 ![](https://www.zhihu.com/equation?tex=%5Cmathbf%7B%5Ctheta%5E%7Bnew%7D%7D+%3D+%5Cmathop%7B%5Carg%5Cmax%7D_%7B%5Ctheta%7D+Q%5Cleft%28+%5Cmathbf%7B%5Ctheta%7D%2C+%5Cmathbf%7B%5Ctheta%5E%7Bold%7D%7D+%5Cright%29) 。

4. 检查对数似然函数或参数是否收敛，如果没收敛，则

![](https://www.zhihu.com/equation?tex=%5Cmathbf%7B%5Ctheta%5E%7Bold%7D%7D+%5Cleftarrow+%5Cmathbf%7B%5Ctheta%5E%7Bnew%7D%7D%5C%5C)

回到第 2 步继续迭代；

如果已收敛，则得到了模型参数。

EM 算法的每次循环可以使不完全数据集的对数似然函数上升，除非其已收敛到局部最大值 (参考文献 [1], Section9.3, Page440)。

参考文献
----

[1] Christopher M. Bishop. Pattern Recognition and Machine Learning, Springer, 2006.

[2] 李航，统计学习方法，清华大学出版社，2012 年。

写下你的评论...