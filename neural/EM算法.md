<!-- @import "../my-style.less" -->
#　　 EM算法
一般地，用$Y$表示观测随机变量的数据，$Z$表示隐随机变量的数据。$\theta$是需要估计的模型参数。EM算法通过迭代求$L(\theta)=\log P(Y|\theta)$的极大似然估计，每次迭代包含两步：E步，求期望；M步，求极大化。

### 算法步骤：
输入：观测变量数据$Y$，隐变量数据$Z$，联合分布$P(Y,Z|\theta)$，条件分布$P(Z|Y,\theta)$;
输出：模型参数$\theta$。
- 选择参数的初值$\theta^{(0)}$，开始迭代；
- E步：记$\theta^{(i)}$为第i次迭代参数$\theta$的估计值，在第i+1次迭代的E步，计算
$$\begin{aligned}
Q(\theta,\theta^{(i)})&=E_z[\log P(Y,Z|\theta)|Y,\theta^{(i)}]\\
&=\sum_z\log P(Y,Z|\theta)P(Z|Y,\theta^{(i)})
\end{aligned}\tag{1}
$$
这里$P(Z|Y,\theta^{(i)})$是在给定观测数据$Y$和当前的参数估计$\theta^{(i)}$下隐变量数据$Z$的条件概率分布；
- M步：求使$Q(\theta,\theta^{(i)})$极大化的$\theta$，确定第i+1次迭代的参数的估计值$\theta^{(i+1)}$
$$
\theta^{(i+1)}=\arg\max_\theta Q(\theta,\theta^{(i)})\tag{2}
$$
- 重复(1)、(2)，直到收敛。