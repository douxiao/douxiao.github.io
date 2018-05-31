---
title: 深度学习——YOLO-v2论文阅读笔记
date: 2018-05-31 09:18:15
tags:
- yolo
- object-detection
categories: 论文阅读
---
<font size="5" color="lightseagreen"> 2016年12月2５，时隔一年，作者更新了yolo升级到了v2版，这篇博客是总结YOLO-v2。
 </font>
<!--more-->

# 一、背景介绍

　　为了提高物体的**定位精准性和召回率**，YOLO作者改进了神经网络提出了YOLO-v2版本，提高了训练图像的分辨率，引入了faster rcnn中的anchor box的思想，对网络结构即各层的设计进行了改进输出层使用卷积层替代YOLO-V1版本的全连接层，采用wordtree的方法，联合使用coco数据集和imagenet数据集训练模型，使之可以实时识别9000类物体。在**识别精度、速度和定位准确度**等方面有大大的提升。
　　该模型有以下几点值得参考
  
　　　1. 对模型的一系列分析和改进；
　　　2. 采用多尺度训练方法，是模型可以适应不同的输入尺寸；
　　　3. 使用wordtree方法值得参考；
　　　4. 端对端的识别训练方法。

　　YOLO9000是可以检测超过9000种类别的实时检测系统。
　　首先，作者在YOLO基础上进行了一系列的改进，产生了YOLOv2。YOLOv2在PASCAL VOC和COCO数据集上获得了目前最好的结果（state of the art）。
　　然后，采用多尺度训练方法，YOLO-v2可以根据速度和精确度需求调整输入尺寸。67FPS时，YOLOv2在VOC2007数据集上可以达到76.8mAP；40FPS，可以达到78.6mAP，比目前最好的Faster R-CNN和SSD精确度更高，检测速度更快。
　　最后提出了目标检测和分类的共训练方法。采用该方法，作者分别在COCO目标检测数据集和ImageNet分类数据集上训练了YOLO9000。联合训练使YOLO9000可以预测没有labelled的目标检测数据。YOLO9000在ImageNet验证集（200类）上获得了19.7mAP。其中，156类没有出现在COCO训练集中，YOLO9000获得了16.0mAP。YOLO9000可以实时识别超过9000类别。　

# 二、核心思想

   YOLO-v2在YOLO-v1基础上做了以下改进如图1所示。

<div align=center>

![](http://ow7va355d.bkt.clouddn.com/yolo-17.png)

</div>

## <font color=blue size=5> Batch Normalization </font>
　　1、CNN在训练过程中网络每层输入的分布一直在改变, 会使训练过程难度加大，但可以通过normalize每层的输入解决这个问题；
　　2、可以提高模型的收敛速度，减少过拟合，有助于规范化模型，可以在舍弃dropout优化后依然不会过拟合。。
　　3、新的YOLO网络在每一个卷积层后添加batch normalization，通过这一方法，mAP获得了2%的提升。
## <font color=blue size=5> High Resolution Classifier </font>
 　　目前的目标检测方法中，基本上都会使用ImageNet预训练过的模型（classifier）来提取特征，如果用的是AlexNet网络，那么输入图片会被resize到不足256X256，导致分辨率不够高，给检测带来困难。为此，新的YOLO网络把分辨率直接提升到了448X448，这也意味之原有的网络模型必须进行某种调整以适应新的分辨率输入。
　　对于YOLOv2，作者首先对分类网络（自定义的darknet）进行了fine tune，分辨率改成448 * 448，在ImageNet数据集上训练10轮（10 epochs），训练后的网络就可以适应高分辨率的输入了。然后，作者对检测网络部分（也就是后半部分）也进行fine tune。这样通过提升输入的分辨率，mAP获得了4%的提升。
## <font color=blue size=5> Convolutional With Anchor Boxes </font>  
　　YOLO采用全连接层来直接预测bounding boxes，导致丢失较多的空间信息从而定位不准；
　　作者在这一版本中借鉴了Faster R-CNN中的anchor思想，回顾一下，anchor是RNP网络中的一个关键步骤，说的是在卷积特征图上进行滑窗操作，每一个中心可以预测9种不同大小的建议框。看到YOLOv2的这一借鉴;
　　为了引入anchor boxes来预测bounding boxes，作者在网络中果断去掉了全连接层。
　　**剩下的具体怎么操作呢？**
　　首先，作者去掉了后面的一个池化层以确保输出的卷积特征图有更高的分辨率；
   然后，通过缩减网络，让图片输入分辨率为416X416，这一步的目的是为了让后面产生的卷积特征图宽高都为奇数，这样就可以产生一个center cell。作者观察到，大物体通常占据了图像的中间位置， 就可以只用中心的一个cell来预测这些物体的位置，否则就要用中间的4个cell来进行预测，这个技巧可稍稍提升效率。最后，YOLOv2使用了卷积层降采样（factor为32），使得输入卷积网络的416X416图片最终得到13X13的卷积特征图（416/32=13）。
　　加入了anchor boxes后，可以预料到的结果是召回率上升，准确率下降。我们来计算一下，假设每个cell预测9个建议框，那么总共会预测13X13×9 = 1521个boxes，而之前的网络仅仅预测7 X7X2 = 98个boxes。具体数据为：没有anchor boxes，模型recall为81%，mAP为69.5%；加入anchor boxes，模型recall为88%，mAP为69.2%。这样看来，准确率只有小幅度的下降，而召回率则提升了7%，说明可以通过进一步的工作来加强准确率，的确有改进空间。
## <font color=blue size=5> Dimension Clusters（维度聚类） </font>
　　作者在使用anchor的时候遇到了两个问题，**第一个是anchor boxes的宽高维度往往是精选的先验框（hand-picked priors）**，虽说在训练过程中网络也会学习调整boxes的宽高维度，最终得到准确的bounding boxes。**但是，如果一开始就选择了更好的、更有代表性的先验boxes维度，那么网络就更容易学到准确的预测位置。**
　　和以前的精选boxes维度不同，作者使用了K-means聚类方法类训练bounding boxes，可以自动找到更好的boxes宽高维度。**传统的K-means聚类方法使用的是欧氏距离函数，也就意味着较大的boxes会比较小的boxes产生更多的error，聚类结果可能会偏离。**为此，作者采用的评判标准是IOU得分（也就是boxes之间的交集除以并集），这样的话，error就和box的尺度无关了，最终的距离函数为：
  
  <font color=red size=4> d(box, centroid) = 1 - IOU(box, centroid) </font>
  　
　　作者通过改进的K-means对训练集中的boxes进行了聚类，判别标准是平均IOU得分，聚类结果如下图：  
  
<div align=center>

![](http://ow7va355d.bkt.clouddn.com/yolo-18.png)

</div>
  
　　可以看到，平衡复杂度和IOU之后，最终得到k值为5，意味着作者选择了5种大小的box维度来进行定位预测，这与手动精选的box维度不同。结果中扁长的框较少，而瘦高的框更多（这符合行人的特征），这种结论如不通过聚类实验恐怕是发现不了的。
　　当然，作者也做了实验来对比两种策略的优劣，如下图，使用聚类方法，仅仅5种boxes的召回率就和Faster R-CNN的9种相当。**说明K-means方法的引入使得生成的boxes更具有代表性，为后面的检测任务提供了便利。**

## <font color=blue size=5> Direct location prediction（直接位置预测）</font> 
　　那么，作者在使用anchor boxes时发现的**第二个问题就是：模型不稳定，尤其是在早期迭代的时候**。大部分的不稳定现象出现在预测box的 (x,y) 坐标上了。在区域建议网络中，预测 (x,y) 以及 tx，ty 使用的是如下公式：

## <font color=blue size=5> Fine-Grained Features（细粒度特征） </font> 
　　上述网络上的修改使YOLO最终在13X13的特征图上进行预测，虽然这足以胜任大尺度物体的检测，但是用上细粒度特征的话，这可能对小尺度的物体检测有帮助。Faser R-CNN和SSD都在不同层次的特征图上产生区域建议（SSD直接就可看得出来这一点），获得了多尺度的适应性。这里使用了一种不同的方法，简单添加了一个转移层（ passthrough layer），这一层要把浅层特征图（分辨率为26X26，是底层分辨率4倍）连接到深层特征图。
<div align=center>
![](http://ow7va355d.bkt.clouddn.com/yolo-20.jpeg)
</div>
　　这个转移层也就是把高低两种分辨率的特征图做了一次连结，连接方式是叠加特征到不同的通道而不是空间位置，类似于Resnet中的identity mappings。这个方法把26X26X512的特征图连接到了13X13X2048的特征图，这个特征图与原来的特征相连接。YOLO的检测器使用的就是经过扩张的特征图，它可以拥有更好的细粒度特征，使得模型的性能获得了1%的提升。（这段理解的也不是很好，要看到网络结构图才能清楚）

　　补充：关于passthrough layer，具体来说就是特征重排（不涉及到参数学习），前面
26X26X512的特征图使用按行和按列隔行采样的方法，就可以得到4个新的特征图，维度都是13X13X512，然后做concat操作，得到13X13X2048的特征图，将其拼接到后面的层，相当于做了一次特征融合，有利于检测小目标。
## <font color=blue size=5> Multi-Scale Training </font> 
   最初的YOLO输入尺寸为448×448，加入anchor boxes后，输入尺寸为416×416。模型只包含卷积层和pooling 层，因此可以随时改变输入尺寸。
　　作者在训练时，每隔几轮便改变模型输入尺寸，以使模型对不同尺寸图像具有鲁棒性。每个10batches，模型随机选择一种新的输入图像尺寸（320,352,...608，32的倍数，因为模型下采样因子为32），改变模型输入尺寸，继续训练。
　　该训练规则强迫模型取适应不同的输入分辨率。模型对于小尺寸的输入处理速度更快，因此YOLOv2可以按照需求调节速度和准确率。在低分辨率情况下（288×288），YOLOv2可以在保持和Fast R-CNN持平的准确率的情况下，处理速度可以达到90FPS。在高分辨率情况下，YOLOv2在VOC2007数据集上准确率可以达到state of the art（78.6mAP）。

<div align=center>

![](http://ow7va355d.bkt.clouddn.com/yolo-21.jpg)
</div>
## <font color=blue size=5> YOLOv2速度的改进（Faster）</font> 
　　YOLO一向是速度和精度并重，作者为了改善检测速度，也作了一些相关工作。
　　大多数检测网络有赖于VGG-16作为特征提取部分，VGG-16的确是一个强大而准确的分类网络，但是复杂度有些冗余。224X224的图片进行一次前向传播，其卷积层就需要多达306.9亿次浮点数运算。
　　YOLOv2使用的是基于Googlenet的定制网络，比VGG-16更快，一次前向传播仅需85.2亿次运算。可是它的精度要略低于VGG-16，单张224X224取前五个预测概率的对比成绩为88%和90%（低一点点也是可以接受的）。
## <font color=blue size=5> Darknet-19</font> 
　　**YOLOv2使用了一个新的分类网络作为特征提取部分，参考了前人的先进经验，比如类似于VGG，作者使用了较多的3X3卷积核，在每一次池化操作后把通道数翻倍。借鉴了network in network的思想，网络使用了全局平均池化（global average pooling），把1X1的卷积核置于3X3的卷积核之间，用来压缩特征。也用了batch normalization（前面介绍过）稳定模型训练。**
　　最终得出的基础模型就是Darknet-19，如下图，其包含19个卷积层、5个最大值池化层（maxpooling layers ），下图展示网络具体结构。Darknet-19运算次数为55.8亿次，imagenet图片分类top-1准确率72.9%，top-5准确率91.2%。
<div align=center>
![]( http://ow7va355d.bkt.clouddn.com/yolo-21.png)
</div>
## <font color=blue size=5> Training for classification</font> 
　　作者使用Darknet-19在标准1000类的ImageNet上训练了160次，用的随机梯度下降法，starting learning rate 为0.1，polynomial rate decay 为4，weight decay为0.0005 ，momentum 为0.9。训练的时候仍然使用了很多常见的数据扩充方法（data augmentation），包括random crops, rotations, and hue, saturation, and exposure shifts。 （这些训练参数是基于darknet框架，和caffe不尽相同）
　　初始的224X224训练后，作者把分辨率上调到了448X448，然后又训练了10次，学习率调整到了0.001。高分辨率下训练的分类网络在top-1准确率76.5%，top-5准确率93.3%。
## <font color=blue size=5> Training for detection</font> 
　　分类网络训练完后，就该训练检测网络了，作者去掉了原网络最后一个卷积层，转而增加了三个3x3X1024的卷积层（可参考darknet中cfg文件），并且在每一个上述卷积层后面跟一个1X1的卷积层，输出维度是检测所需的数量。对于VOC数据集，预测5种boxes大小，每个box包含5个坐标值和20个类别，所以总共是5X（5+20）= 125个输出维度。同时也添加了转移层（passthrough layer ），从最后那个3X3X512的卷积层连到倒数第二层，使模型有了细粒度特征。
　　作者的检测模型以0.001的初始学习率训练了160次，在60次和90次的时候，学习率减为原来的十分之一。其他的方面，weight decay为0.0005，momentum为0.9，依然使用了类似于Faster-RCNN和SSD的数据扩充（data augmentation）策略。
## <font color=blue size=5> YOLOv2分类的改进（Stronger）</font> 
　　作者提出了将分类数据和检测数据综合的联合训练机制。该机制使用目标检测标签的数据训练模型学习定位目标和检测部分类别的目标；再使用分类标签的数据取扩展模型对多类别的识别能力。在训练的过程中，混合目标检测和分类的数据集。当网络接受目标检测的训练数据，反馈网络采用YOLOv2 loss函数；当网络接受分类训练数据，反馈网络只更新部分网络参数。
　　这类训练方法有一定的难度。目标识别数据集仅包含常见目标和标签（比如狗，船）；分类数据集包含更广和更深的标签。比如狗，ImageNet上包含超过100种的狗的类别。如果要联合训练，需要将这些标签进行合并。
　　大部分分类方法采用softmax输出所有类别的概率。采用softmax的前提假设是类别之间不相互包含（比如，犬和牧羊犬就是相互包含）。因此，我们需要一个多标签的模型来综合数据集，使类别之间不相互包含。
# 三、网络结构


# 四、参考文献

1.　[YOLOv2 论文笔记](https://blog.csdn.net/Jesse_Mx/article/details/53925356)
2.　[YOLO升级版：YOLOv2和YOLO9000解析](https://zhuanlan.zhihu.com/p/25052190)



