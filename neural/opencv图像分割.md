<!-- @import "../my-style.less" -->
<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

* [图像分割](#图像分割)
	* [1. nucleus细胞核分割（分水岭算法）](#1-nucleus细胞核分割分水岭算法)
	* [GrabCut迭代图割](#grabcut迭代图割)

<!-- /code_chunk_output -->

# 图像分割
- 使用分水岭算法进行图像分割
- 使用迭代图割(GrabCut)算法进行彩色图像分割

## 1. nucleus细胞核分割（分水岭算法）

```python
#读取数据
nucleus_dir=os.path.join(HOME,'dataset/nucleus/stage1_train')
image_ids = next(os.walk(nucleus_dir))[1]

#加载图像
idx=np.random.randint(0,len(image_ids))
img=cv2.imread(os.path.join(nucleus_dir,'{}/images/{}.png'.format(image_ids[idx],image_ids[idx])))
imgGray=cv2.cvtColor(img,cv2.COLOR_BGR2GRAY)
fig=plt.figure(figsize=(16,16))
plt.subplot(121)
plt.imshow(img[:,:,::-1])
plt.xticks([]),plt.yticks([])
plt.title('原图')
plt.subplot(122)
plt.imshow(imgGray, cmap='gray')
plt.xticks([]),plt.yticks([])
plt.title('灰度')
```
![img _segment](/assets/img_segment_1.png)

接下来使用cv2.threshold进行图像二值化处理。
```python
#使用THRESH_OTSU二值化双峰图像
ret, thresh=cv2.threshold(imgGray, 0, 255, cv2.THRESH_BINARY_INV+cv2.THRESH_OTSU)
plt.figure(figsize=(8,8))
plt.imshow(thresh, cmap='gray')
```
![img_segment_2](/assets/img_segment_2.png)
opencv中使用threshold(src, thresh, maxval, type[, dst]) -> retval, dst可以对src数组进行阈值处理。
type类型有：
|type|效果|
|:-:|:-:|
|THRESH_BINARY|超过thresh的像素被设为maxval, 其他被设为0|
|THRESH_BINARY_INV|和THRESH_BINARY作用相反|
|THRESH_TRUNC|超过thresh的像素被设为maxval，其他像素保持不变|
|THRESH_TOZERO|超过thresh的像素不变, 其他设为0|
|THRESH_TOZERO_INV|和THRESH_TOZERO作用相反|
|THRESH_OTSU|Ostu算法，从图像灰度直方图的双峰中自动选择阈值，可以与前面的type组合|
THRESH_TRIANGLE|Triangle算法，可以与前面的type组合|

```python
#对图像进行开运算，先腐蚀再膨胀，通常用来去除小的噪点
kernel=np.ones((3,3), np.uint8)
opening=cv2.morphologyEx(thresh,cv2.MORPH_OPEN,kernel,iterations=2)
#对开运算的结果进行膨胀，得到大部分都是背景的区域
sure_bg=cv2.dilate(opening, kernel, iterations=3)
plt.figure(figsize=(16,16))
plt.subplot(221)
plt.imshow(sure_bg, cmap='gray')
plt.xticks([]),plt.yticks([])
plt.title('sure_bg')
#通过distanceTransform获取前景区域
dist_transform=cv2.distanceTransform(opening, cv2.DIST_L2, 5)
plt.subplot(222)
plt.imshow(dist_transform, cmap='gray')
plt.xticks([]),plt.yticks([])
plt.title('dist_transform')
ret, sure_fg=cv2.threshold(dist_transform, 0.1*dist_transform.max(), 255, 0)
plt.subplot(223)
plt.imshow(sure_fg, cmap='gray')
plt.xticks([]),plt.yticks([])
plt.title('sure_fg')
#sure_bg与sure_fg相减，得到既有前景又有背景的重合区域
sure_fg=np.uint8(sure_fg)
unknown=cv2.subtract(sure_bg, sure_fg)
#连通区域处理
ret,markers=cv2.connectedComponents(sure_fg)
markers=markers+1
markers[unknown==255]=0
#分水岭算法
markers=cv2.watershed(img, markers)
img[markers==-1]=[255,0,0]
plt.subplot(224)
plt.imshow(img,cmap='jet')
plt.xticks([]),plt.yticks([])
plt.title('seg_img')
```
![img_segment_3](/assets/img_segment_3.png)
上述代码中：
- cv2.distanceTransform(src, distanceType, maskSize[, dst[, dstType]]) -> dst:
针对二值化图像进行空间点对目标点的距离计算，最终将二值化图像转变为灰度图像，图像中每个像素点的灰度值是该点到距离最近的背景点的距离。
- cv2.connectedComponents(image[, labels[, connectivity[, ltype]]]) -> retval, labels:
连通区域计算，connectivity可以选择4或者8，分别表示相邻和相邻+对角线相邻的属于一个区域。
- cv2.watershed(image, markers) -> markers
使用分水岭，标记图像会被修改，边界区域会被标记成-1。

## GrabCut迭代图割
```python
img=cv2.imread(os.path.join(DATA_DIR,'style/lion2.jpeg'))
plt.figure(figsize=(8,12))
plt.imshow(img[:,:,::-1])
# plt.xticks([]),plt.yticks([])
plt.title('原图')

#创建掩模、背景图和前景图
mask=np.zeros(img.shape[:2], np.uint8)
bgdModel=np.zeros((1,65), np.float64)
fgdModel=np.zeros((1,65),np.float64)

#初始化矩形区域，这个矩形必须完全包含前景
rect=(50,50,1200,2000)

#GrabCut算法，迭代5次
cv2.grabCut(img, mask, rect, bgdModel, fgdModel, 5, cv2.GC_INIT_WITH_RECT)

#mask中，值为2和0的统一转化为0, 1和3转化为1 
print(mask)
mask2=np.where((mask==2)|(mask==0),0,1).astype(np.uint8)
reg_img=img*mask2[:,:,np.newaxis]

plt.figure(figsize=(8,8))
plt.imshow(reg_img[:,:,::-1])
# plt.xticks([]),plt.yticks([])
plt.title('分割后')
```
![img_segment_4](/assets/img_segment_4.png)
![img_segment_5](/assets/img_segment_5.png)