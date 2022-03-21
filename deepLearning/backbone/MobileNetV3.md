MobileNetV3是该系列第三代的网络架构，作者研究了自动搜索算法和网络设计相结合的办法，提出了 MobileNetV3_Large 和 MobileNetV3_Small 用于不同硬件条件。
## 回顾

MobileNetV1 引入了深度可分离卷积(depthwise separable convolutions)替代普通的卷积网络层。深度可分离卷积包含两个独立的层：轻量级的深度卷积和重量级的点卷积(pointwise convolutions)。这种卷积减少FLOPS和模型大小，使得模型速度更快，计算开销更小，且有着不错的准确率。结构对比如下：
![mobilenetv1](../img/mobilenetv1.png)

MobileNetV2 引入bottleneck结构和残差单元，并在残差结构中引入卷积层过滤器扩展(expansion rate)和轻量级的深度卷积。如下图所示对比了 resnet 和 MobileNetV2 残差结构：
![mobilenetv2](../img/mobilenetv2.png)

## MnasNet

MnasNet在bottleneck结构中引入了基于 [squeeze and excitation](http://openaccess.thecvf.com/content_cvpr_2018/papers/Hu_Squeeze-and-Excitation_Networks_CVPR_2018_paper.pdf) 的轻量级注意力模块和用于自动构造神经网络的NAS(neural architecture search)模块。

## Network Search，NetAdapt

作者使用基于平台的网络结构搜索模块来生成网络的block。这个模块和 MnasNet 的 NAS 模块完全相同，因此将 MnasNet-A1 作为搜索所得的模型。将 NAS 得到的模型通过 NetAdapt 按层精确调校，NetAdapt提出一系列proposals，选取最大化$\frac{\Delta Acc}{\Delta latency}$的结构。持续这个过程直到latency达到要求。

## swish激活函数

通过在MobileNetV3中引入swish激活函数可以明显提高神经网络的准确率，定义如下：$$swish x=x*\delta(x)$$
但是这个函数大幅增加了手机设备的计算量，原因是sigmoid函数在手机设备上的计算代价更大。
为此使用类似于sigmoid的$\frac{ReLU6(x+3)}{6}$替代。swish 公式变成如下：
$$h-swish[x]=x\frac{ReLU6(x+3)}{6}$$
同时随着网络层次的深入，应用激活函数的代价会下降。因此，作者只在网络深层应用h-swish函数。
