# SSD(a single-shot detector)目标检测

- 引入SSD，比之前最好的目标检测网络YOLO更快，更准确。它的准确率比之Faster RCNN也比逊色。
- SSD的核心是通过特征图的小型卷积过滤来预测分类分数和相对固定bounding box的偏移box。
- 为了更高的检测准确率，我们从不同尺寸的特征图中生成不同尺寸的预测，并通过长宽比明确的分离这些预测。
- 这些设计特点使模型可以简单的端到端训练并且具有高准确率，即使是在低分辨率的输入照片中，在保持准确率的同事进一步提高了检测速度。
- 实验评估了PASCAL VOC, COCO和ILSVRC数据集的不同输入尺寸下模型的速度和准确率，并和最近最好的目标检测技术进行了对比。


## SSD模型结构

SSD是通过前馈卷积网络产生固定尺寸的bounding box和对应的分类分数，然后使用NMS(non-maximum suppression)产生最后的检测目标。网络的前面部分通常使用用于分类的基础网络(backbone)比如VGG,Resnet,Mobilenet等。网络的后面部分是辅助网络结构用于产生检测目标和以下特征：
- 多尺寸特征图，用于目标检测；
- 卷积预测器用于目标检测；
- 预选的boxes和长宽比。