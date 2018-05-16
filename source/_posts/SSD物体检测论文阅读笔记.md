---
title: 深度学习——SSD论文阅读笔记
date: 2018-05-14 08:09:01
tags:
- SSD
- 目标检测
categories: 论文阅读
---
<font size="5" color="lightseagreen"> 　　这篇文章是总结SSD算法，其英文的全名是Single Shot MultiBox Detector，Single Shot 代表了给算法属于one-stage方法，MultiBox指的是多边框预测，本文详细的介绍了SSD的工作原理，结构框架和与其他网络比如VGG,YOLO,R-CNN等网络对比。
 </font>
<!--more-->

# 一、背景介绍

　　基于深度学习的目标检测方法主要有两种**（1）two-stage方法：R-CNN系列，其主要思路是先通过启发式方法（selective search）或者CNN网络（RPN）产生一系列的候选框，然后对这些候选框进行分类和回归，two-stage方法的优势是准确率高；（2）one-stage方法是，如YOLO和SSD,其主要思路是均匀的在图片的不同位置进行密集的抽样，抽样是采用不同尺度和长宽比，然后利用CNN提取特征后直接进行分类和回归，整个过程只需要一步。**速度明显提高。
　　本文主要介绍one-stage方法。**以下是YOLO和SSD的对比**：
　　YOLO能够达到实时的效果，但是缺点还是比较明显的: 
　　　　1）一个网格只能够预测两个物体；
　　　　2）对物体的尺度比较敏感，对尺度变化较大的物体泛化能力较差；
　　　　3）难以检测小目标，而且定位不准。
　　**针对YOLO的不足**，**SSD**同时兼顾速度和mAP,两者不同点,换句话说也是**改进点**：
　　　　1）SSD采用CNN直接进行检测，然而YOLO在全连接层之后进行检测；
　　　　2）SSD提取的不同尺度的feature map来做检测，大尺度的特征图（靠前的特征图）用来检测比较小的物体，而小尺度的特征图用来检测大物体；
　　　　3）SSD采用了不同尺度和不同长宽比的先验框（Prior Box ，Default Box，在faster r-cnn中叫做锚，Anchor）。


# 二、核心算法

## 2.1 多尺度的Feature Map

　　作者同时采用低层和高层的不同尺度的feature map做检测。
　　所谓多尺度采用大小不同的特征图，CNN网络一般前面的特征图比较大，后面会逐渐采用stride=2的卷积或者pool来降低特征图大小，这正如图3所示，**一个比较大的特征图和一个比较小的特征图，它们都用来做检测。**这样做的好处是比较大的特征图来用来检测相对较小的目标，而小的特征图负责检测大目标，如图4所示，8x8的特征图可以划分更多的单元，但是其每个单元的先验框尺度比较小。
<div align=center>
 ![](http://ow7va355d.bkt.clouddn.com/feature_map1.jpg)
</div>  
  
## 2.2 Default box (prior box)  

　　**Default box**，是指在feature map的每个小格(cell)上都有一系列固定大小的box，如下图有4个（下图中的虚线框，仔细看格子的中间有比格子还小的一个box）。
　　所以假设每个feature map包含:
　　1) mxn个cell;
　　2) 每个cell预测k个default box
　　3)每个default box有 c个类别+4个offset(坐标偏移量)
　　综上每个feature map的输出是：**k·(c+4)·(mxn)**。
<div align=center>
![](http://ow7va355d.bkt.clouddn.com/ssd1.jpg)
</div> 

　　**训练中还有一个东西：prior box**，是指实际中选择的default box（每一个feature map cell 不是k个default box都取）。**也就是说default box是一种概念，prior box则是实际的选取**。
　　训练中一张完整的图片送进网络获得各个feature map，对于**正样本训练**来说，需要先将prior box与ground truth box做匹配，匹配成功说明这个prior box所包含的是个目标，但离完整目标的ground truth box还有段距离，训练的目的是保证default box的分类confidence的同时将prior box尽可能回归到ground truth box。 
　　**举个列子：假设一个训练样本中有2个ground truth box，所有的feature map中获取的　prior box一共有8732个。那个可能分别有10、20个prior box能分别与这2个ground truth box匹配上。训练的损失包含定位损失和回归损失两部分。**
　　这里用到的 default box 和Faster RCNN中的 anchor 很像，在Faster RCNN中 anchor 只用在最后一个卷积层，但是在本文中，default box 是应用在多个不同层的feature map上。
## 2.3多尺度feature map 和default box 图解

 　　**多尺度 feature map 得到的 default box及其四个位置的偏移和21个类别的置信度**对于不同尺度的fature map( 38x38x512  19x19x1024 10x10x512 5x5x256 3x3x256 1x1x256 )
　　**以5x5x256,default box=6为例**
<div align=center>
![](http://ow7va355d.bkt.clouddn.com/ssd3.jpg)

![](http://ow7va355d.bkt.clouddn.com/ssd5.jpg)
</div>

　　**按照不同的 scale 和 ratio 生成，k 个 default boxes，这种结构有点类似于 Faster R-CNN 中的 Anchor。(此处k=6所以：5x5x3=45 boxes)**

<div align=center>
 ![](http://ow7va355d.bkt.clouddn.com/ssd6.jpg)
 ![](http://ow7va355d.bkt.clouddn.com/ssd13.jpg)
 ![](http://ow7va355d.bkt.clouddn.com/ssd7.jpg)
</div>

## 2.4 Default box的scale和aspect ratiodio定义方法
  
　　那么default box的scale（大小）和aspect ratio（横纵比）要怎么定呢？假设我们用m个feature maps做预测，那么对于每个featuer map而言其default box的scale是按以下公式计算的： 
<div align=center>
![](http://ow7va355d.bkt.clouddn.com/ssd-defaultbox.png)
</div>

　　每一个 default box 的中心，设置为：(i+0.5/|fk|,j+0.5/|fk|)，其中，|fk| 是第 k 个 feature map 的大小，同时，i,j∈[0,|fk|)。
　　可以看出这种**default box在不同的feature层有不同的scale**，**在同一个feature层又有不同的aspect ratio**，因此基本上可以覆盖输入图像中的各种形状和大小的object！

## 2.4 正负样本
　　
　　将**prior box **和 **grount truth box** 按照IOU（JaccardOverlap）进行匹配，匹配成功则这个prior box就是positive example（正样本），如果匹配不上，就是negative example（负样本），显然这样产生的负样本的数量要远远多于正样本。这里将前向loss进行排序，选择最高的num_sel个prior box序号集合 D。那么如果Match成功后的正样本序号集合P。那么最后正样本集为 P−D∩P，负样本集为 D−D∩P。同时可以通过规范num_sel的数量（是正样本数量的三倍）来控制使得最后正、负样本的比例在 1：3 左右。
　　在结合 feature maps 上，所有不同尺度、不同 aspect ratios 的 default boxes，它们预测的 predictions 之后。可以想见，我们有许多个 predictions，包含了物体的不同尺寸、形状。如下图，狗狗的 ground truth box 与 4×4 feature map 中的红色 box 吻合，所以其余的 boxes 都看作负样本。 

<div align=center>
 ![](http://ow7va355d.bkt.clouddn.com/ssd12.png)
</div>

　　在生成一系列的 predictions 之后，会产生很多个符合 ground truth box 的 predictions boxes，但同时，不符合 ground truth boxes 也很多，而且这个 negative boxes，远多于 positive boxes。这会造成 negative boxes、positive boxes 之间的不均衡。训练时难以收敛。
　　因此，本文采取，先将每一个物体位置上对应 predictions（default boxes）是 negative 的 boxes 进行排序，按照 default boxes 的 confidence 的大小。 选择最高的几个，保证最后 negatives、positives 的比例在 3:1。本文通过实验发现，这样的比例可以更快的优化，训练也更稳定。 

## 2.5 Data augmentation

　　本文同时对训练数据做了 data augmentation，数据增广。
　　每一张训练图像，随机的进行如下几种选择：
　　１）使用原始的图像
　　２）随机采样多个 patch(CropImage)，与物体之间最小的 jaccard overlap 为：0.1，0.3，0.5，0.7 与 0.9
　　采样的 patch 是原始图像大小比例是 [0.3，1.0]，aspect ratio 在 0.5 或 2。
　　当 groundtruth box 的 中心（center）在采样的 patch 中且在采样的 patch中 groundtruth box面积大于0时，我们保留CropImage。
　　在这些采样步骤之后，每一个采样的 patch 被 resize 到固定的大小，并且以 0.5 的概率随机的 水平翻转（horizontally flipped，翻转不翻转看prototxt，默认不翻转）
　　这样一个样本被诸多batch_sampler采样器采样后会生成多个候选样本，然后从中随机选一个样本送人网络训练。

## 2.6 Loss Function损失函数

　　**损失函数方面：**和Faster RCNN的基本一样，由分类和回归两部分组成，可以参考Faster RCNN，这里不细讲。总之，回归部分的loss是希望预测的box和prior box的差距尽可能跟ground truth和prior box的差距接近，这样预测的box就能尽量和ground truth一样。
<div align=center>
 ![](http://ow7va355d.bkt.clouddn.com/ssd8.png)
</div>

　　上面得到的8732个目标框经过Jaccard Overlap筛选剩下几个了；其中不满足的框标记为负数，其余留下的标为正数框。紧随其后：
<div align=center>
![](http://ow7va355d.bkt.clouddn.com/ssd9.png)
</div>

　　1. N 是与 ground truth box 相匹配的 prior boxes 个数;
　　2. localization loss（loc） 是 Fast R-CNN 中 Smooth L1 Loss，用在 predict box（l） 与 ground truth box（g） 参数（即中心坐标位置，width、height）中，回归 bounding boxes 的中心位置，
以及 width、height;
　　3. confidence loss（conf） 是 Softmax Loss，输入为每一类的置信度 c;
　　4. 权重项 α，可在protxt中设置 loc_weight，默认设置为1。



# 三、网络结构

## 3.1整体网络结构

该论文的网络结构实在VGG16的基础上改进的.
如下图：
　　　1)使用5个conv stage;
　　　2)用了astrous算法将FC6、FC7两个卷积层转化成为两个卷积层；
　　　3)在额外增加3个卷积层和一个average pooling层；
　　　4)不同层次的feature map 分别用于default box的偏移和不同类别的得分预测；
  
**优点：**
　　1)增加的feature map的大小变化比较大，允许能够检测不同尺度下的物体。
　　2)低层的feature map感受野比较小,高层的感受野比较大，在不同的feature map下进行卷积，可以达到多尺度的目的。
　　3)相比YOLO后面存在两个全连接层，全连接层以后的输出都会观察到整幅图像，然而SSD去掉了全连接层,每个输出只会感受到目标周围的信息,包括上下文，并且用了不同的feature map 预测不同的长宽比。这样比YOLO增加了预测更多的box。
   
　　算法的主网络结构是VGG16，将最后两个全连接层改成卷积层，并随后增加了4个卷积层来构造网络结构。对其中5种不同的卷积层的输出（feature map）分别用两个不同的 3×3 的卷积核进行卷积，一个输出分类用的confidence，每个default box 生成21个类别confidence；一个输出回归用的 localization，每个 default box 生成4个坐标值（x, y, w, h）。此外，这5个feature map还经过 PriorBox 层生成 prior box（生成的是坐标）。上述5个feature map中每一层的default box的数量是给定的(8732个)。最后将前面三个计算结果分别合并然后传给loss层。

<div align=center>
![](http://ow7va355d.bkt.clouddn.com/ssd10.png)
</div>

更具体的框架如下：

<div align=center>
![](http://ow7va355d.bkt.clouddn.com/ssd11.png)
</div>


# 四、参考文献

1. [SSD](https://zhuanlan.zhihu.com/p/24954433)
2. [论文阅读：SSD: Single Shot MultiBox Detector](https://blog.csdn.net/u010167269/article/details/52563573)
3. [SSD详解Default box的解读](https://blog.csdn.net/wfei101/article/details/78597442)
4. [深度学习笔记（七）SSD论文阅读笔记](https://www.cnblogs.com/xuanyuyt/p/7447111.html)











