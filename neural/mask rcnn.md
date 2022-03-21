# read about mask-rcnn

## RPN Network实现

- 通过backbone（如resnet50,resnet101等）的不同网络层次，自底向上得到图像的特征图谱，命名为C1,C2,C3,C4,C5。然后再由网络层次自顶向下，侧向结合C2,C3,C4,C5，得到P5,P4,P3,P2,其中P5= conv_PyramidSize_1(C5)，然后对P5,P4,P3,P2分别使用conv_PyramidSize_3的卷积层得到pyramid feature map，并对P5下采样maxpool_2得到P6用于后续的RPN model建立。
- 通过 **get_anchors(self, image_shape)** anchor计算。将不同尺寸的anchor（如{32,64,128,256,512}）分别对应到P2,P3,P4,P5,P6，这样只需要再加上[0.5,1,2]三种ratio的anchor，即可以得到image上所有的anchor，最后对所有的anchor执行normalize。
- 通过 **build_rpn_model(anchor_stride, anchors_per_location, depth)** 建立RPN model：将不同尺寸的 pyramid feature map（如{256\*256,128\*128,64\*64,32\*32,16\*16}）首先通过 **conv_512_3_relu** 得到不同的map，将map通过 conv_6_1_linear 得到 rpn_class_logits ，再将 rpn_class_logits 通过 **softmax** 激活函数层得到 rpn_class_probs （size=[batch,num_anchors_of_image,2])，其中 rpn_class_probs[:,:,0]表示背景分， rpn_class_probs[:,:,1]表示前景分，值范围(0-1)。anchor的前景分越大表示存在目标的概率越大。将map通过 **conv_12_1_linear** 得到 rpn_class_bbox(deltas) （size=[batch,num_anchors_of_image, (dy,dx,log(dh),log(dw)]）。
- **ProposalLayer**从所有bounding box中选取出前景概率较大 bounding box proposals
    - 将 rpn_class_probs 由大到小排列，获最大的**PRE_NMS_LIMIT**条probs。并由此计算相应的 rpn_class_bbox，再除与**RPN_BBOX_STD_DEV**之后（这里的暂时不清楚什么原因，猜测是根据faster rcnn RPN的损失函数$$L(\{p_{i}\},\{t_{i}\})=\frac{1}{N_{cls}}\sum_{i}L_{cls}(p_{i},p_{i}^*)+\lambda\frac{1}{N_{reg}}\sum_{i}p_{i}^*L_{reg}(t_{i},t_{i}^*)$$中的$\lambda$有关）和anchors；
    - 将 rpn_class_bbox 和 anchors 代入 **apply_box_deltas_graph(boxes, deltas)**得到修正后的 anchors；
    - 将 refined anchors 裁剪 **clip_boxes_graph(boxes, window)** ，然后 **tf.image.non_max_suppression** 过滤得到前景概率大于0.7的所有 proposals ，并在需要的时候对 proposals 进行补0操作，构成需要的 proposal 数量。

## 生成detection targets


通过**DetectionTargetLayer**层得到用于后续训练的目标的box,ids以及对应的proposals等。将RPN网络生成的proposals 和 ground truth class ids, gound truth box, gound truth box mask代入**detection_targets_graph(proposals, gt_class_ids, gt_boxes, gt_masks, config)**函数中，得到用于后续计算的检测目标 ROIs , class IDs, target deltas , target mask。
- 该函数首先去除了各个参数的zero padding，然后将id,boxes,masks等分离为crowd和no-crowd两种
- 然后计算proposals和gound truth boxes的交并集**overlaps_graph**，得到 overlaps 和 crowd_overlaps ，将overlaps中IoU>=0.5的作为 positive_indices,将overlaps中IoU<0.5且crowd_iou_max < 0.001的作为negative_indices
- 然后在正负样本中选取指定数量(TRAIN_ROIS_PER_IMAGE)的proposals rois，并保持一定的正负比例(1/3)。
- 使用**box_refinement_graph(box, gt_box)**将正样本positive_rois和roi_gt_box代入，得到从roi转变到gt_box的refine过程deltas[dy, dx, log(dh), log(dw)],而后再除与**BBOX_STD_DEV** (这里的作用应该等同于前面的RPN_BBOX_STD_DEV) 精简到标准差形式用于后续的检测。
- 接着根据positive_rois和**tf.image.crop_and_resize**函数，将positvie_rois_masks代入，可以裁剪到rois尺寸，并利用bilinear插值法伸缩到固定的尺寸的新positvie_rois_masks目标。
- 最终 
    - rois=postive_rois+negative_rois+P(padding)；
    - roi_gt_class_ids=positve_rois_class_ids+N(negative, zero padding)+P(zero padding)；
    - deltas=positve_rois_gt_box_refine+N(zero padding)+P(zero padding)： anchor和ground truth bounding box的偏移；
    - masks=positive_rois_mask+N(zero padding)+P(zero padding)。

## 网络头部：将网络分为classifier_graph和mask_graph
 
### classifier_graph头部

根据前面的 rois, mrcnn_feature_maps,输入数据input_image_meta等，代入函数**fpn_classifier_graph(rois, feature_maps, image_meta,...)**中：
- 首先将rois通过**PyramidROIAlign**,这个函数中将不同的rois根据级别对应到不同的feature_map中(如rois的{32,64,128,256,512}对应P2,P3,P4,P5)，然后利用tf.image.crop_and_resize将不同的box裁剪出的子feature maps伸缩到固定的尺寸。将这些pooled maps 根据pyramid级别（P2->P3->P4->P5）排序。
- 将这些pooled maps通过两个1024 FC layers，其中包括batch norm 和 relu activate.
- 再通过两个并列的head，一个用来分类（dense->softmax），一个用来得出bbox的deltas(dense_linear)。
- 输出 mrcnn_class_logits ， mrcnn_class ， mrcnn_bbox 预测和anchor的偏移

### mask_graph头部

根据 rois, mrcnn_feature_map等数据，代入函数**build_fpn_mask_graph(rois, feature_maps, image_meta,...)**中：
- 首先同样是将rois通过**PyramidROIAlign**,得到不同的子 feature maps，输出size=[batch, num_boxes, pool_height, pool_width, channels]；
- 再通过四个conv_256_3的卷积层，其中包含batch_norm和relu activate，接着通过convtranspose_256_2_2_relu和conv_NumClasses_1_1_sigmoid得到size=[batch, num_boxes, pool_height, pool_width, num_classes],值介于0-1之间的mask（1表示mask边缘?）。

## 损失函数

- rpn_class_loss：根据输入的 input_rpn_match 概率(0 neutral,1 positive,-1 negative)和 rpn_class_logits，代入 **rpn_class_loss_graph(rpn_match, rpn_class_logits)** 得到。
- rpn_bbox_loss_graph：根据输入的 input_rpn_bbox ， input_rpn_match 和 rpn_class_bbox，代入 **rpn_bbox_loss_graph(config, target_bbox, rpn_match, rpn_bbox)** 得到。
- class_loss： target_class_ids ， mrcnn_class_logits ， active_class_ids（数据集的class IDs组合）代入 **mrcnn_class_loss_graph(target_class_ids, pred_class_logits,active_class_ids)** 得到。
- bbox_loss： target_bbox, target_class_ids, mrcnn_bbox 代入 **mrcnn_bbox_loss_graph(target_bbox, target_class_ids, pred_bbox)**。
- mask_loss: target_mask, target_class_ids, mrcnn_mask 代入 **mrcnn_mask_loss_graph(target_masks, target_class_ids, pred_masks)**。