---
title: 深度学习——YOLO-v3论文阅读笔记
date: 2018-06-01 08:18:21
tags:
- yolo
- object-detection
categories: 论文阅读
---
<font size="5" color="lightseagreen"> yolo作者更新了yolo升级到了yolo-v3版本，这篇博客是总结yolo-v3
 </font>
<!--more-->

# 一、背景介绍

　　在做背景介绍前先看一张图，YOLOs-v3与目前最好的实时检测网络的性能对比图。

<div align=center>
![](http://ow7va355d.bkt.clouddn.com/yolo21.jpeg)
</div>

　　YOLO作者推出 YOLOv3版，在Titan X上训练时，在mAP相当的情况下，v3的速度比 RetinaNet快3.8倍，同时YOLOv3 可以在22ms之内执行完一张320×320的图片，mAP得分是 51.5，和SSD的准确率相当，但是比它快三倍。YOLOv3非常快速和准确，在IoU=0.5的情况下，与Focal Loss的mAP值相当，但快了4倍。
　　YOLO的V1和V2都不如SSD的算法，主要原因是V1的448尺寸和V2版本的416尺寸都不如SSD的300，以上结论都是实验测试的，V3版本的416应该比SSD512好，可见其性能。
  
  
# 二、核心思想

## 2.1 基本思想

　　首先通过特征提取网络对输入的图像提取特征，得到一定大小的feature map 比如 13X13，然后将输入的图像分为13X13个grid cell，然后如果groundtruth中的某个物体的中心坐标落到那个grid cell中就由该grid cell预测该物体,**每个grid cell 都会预测固定数量的bounding box (v1 :2个，V2 ：5个 ，V3 : 3个)**这几个bounding box的初始size是不一样的），那么这几个bounding box中最终是由哪一个来预测该object？**答案是：这几个bounding box中只有和ground truth的IOU最大的bounding box才是用来预测该物体。**


## 2.2 bounding box坐标预测

　　简单讲就是下面这个截图的公式，tx、ty、tw、th就是模型的预测输出。cx和cy表示grid cell的坐标，比如某层的feature map大小是13X13，那么grid cell就有13*13个，第0行第1列的grid cell的坐标cx就是0，cy就是1。pw和ph表示预测前bounding box的size。bx、by。bw和bh就是预测得到的bounding box的中心的坐标和size。坐标的损失采用的是平方误差损失。 

![](http://ow7va355d.bkt.clouddn.com/yolo%20-22.jpeg)

![](http://ow7va355d.bkt.clouddn.com/162f2b2f82d0e5cb)
　　得出的预测 bw 和 bh 使用图像的高和宽进行归一化。即，如果包含目标（狗）的框的预测 bx 和 by 是 (0.3, 0.8)，那么 13 x 13 特征图的实际宽和高是 (13 x 0.3, 13 x 0.8)。

## 2.3 多标签分类

　　因此网络结构上就将原来用于单标签多分类的softmax层换成用于多标签多分类的逻辑回归层。首先说明一下为什么要做这样的修改，原来分类网络中的softmax层都是假设一张图像或一个object只属于一个类别，但是在一些复杂场景下，一个object可能属于多个类，比如你的类别中有woman和person这两个类，那么如果一张图像中有一个woman，那么你检测的结果中类别标签就要同时有woman和person两个类，这就是多标签分类，需要用逻辑回归层来对每个类别做二分类。逻辑回归层主要用到sigmoid函数，该函数可以将输入约束在0到1的范围内，因此当一张图像经过特征提取后的某一类输出经过sigmoid函数约束后如果大于0.5，就表示属于该类。

## 2.4 多scale融合

　　原来的YOLO v2有一个层叫：passthrough layer，假设最后提取的feature map的size是13X13，那么这个层的作用就是将前面一层的26X26的feature map和本层的13X13的feature map进行连接，有点像ResNet。
　　当时这么操作也是为了加强YOLO算法对小目标检测的精确度。这个思想在YOLO v3中得到了进一步加强，在YOLO v3中采用类似FPN的upsample和融合做法（最后融合了3个scale，其他两个scale的大小分别是26X26和52X52），在多个scale的feature map上做检测，对于小目标的检测效果提升还是比较明显的。前面提到过在YOLO v3中每个grid cell预测3个bounding box，看起来比YOLO v2中每个grid cell预测5个bounding box要少，其实不是！因为YOLO v3采用了多个scale的特征融合，所以boundign box的数量要比之前多很多，以输入图像为416X416为例：(13X13+26X26+52X52)X3和13X13X5相比哪个更多应该很清晰了。
　　YOLO v3 在 3 个不同尺度上进行预测。检测层用于在三个不同大小的特征图上执行预测，特征图步幅分别是 32、16、8。这意味着，当输入图像大小是 416 x 416 时，我们在尺度 13 x 13、26 x 26 和 52 x 52 上执行检测。
　　该网络在第一个检测层之前对输入图像执行下采样，检测层使用步幅为 32 的层的特征图执行检测。随后在执行因子为 2 的上采样后，并与前一个层的特征图（特征图大小相同）拼接。另一个检测在步幅为 16 的层中执行。重复同样的上采样步骤，最后一个检测在步幅为 8 的层中执行。
　　在每个尺度上，每个单元使用3个锚点预测3个边界框，锚点的总数为9（不同尺度的锚点不同）。
<div align=center>  
![](http://ow7va355d.bkt.clouddn.com/yolo-25)
</div>
   
　　作者称这帮助 YOLO v3 在检测较小目标时取得更好的性能，而这正是 YOLO 之前版本经常被抱怨的地方。上采样可以帮助该网络学习细粒度特征，帮助检测较小目标。　

## 2.5 输出处理

　　对于大小为 416 x 416 的图像，YOLO 预测 ((52 x 52) + (26 x 26) + 13 x 13)) x 3 = 10647 个边界框。但是，我们的示例中只有一个对象——一只狗。那么我们怎么才能将检测次数从 10647 减少到 1 呢？
　　目标置信度阈值：首先，我们根据它们的 objectness 分数过滤边界框。通常，分数低于阈值的边界框会被忽略。
　　非极大值抑制：非极大值抑制（NMS）可解决对同一个图像的多次检测的问题。例如，上图红色网格单元的 3 个边界框可以检测一个框，或者临近网格可检测相同对象。
<div align=center>  
![](http://ow7va355d.bkt.clouddn.com/yolo-26)
</div>


## 2.6 Darknet-53
　　一方面基本采用全卷积（YOLO v2中采用pooling层做feature map的sample，这里都换成卷积层来做了），另一方面引入了residual结构（YOLO v2中还是类似VGG那样直筒型的网络结构，层数太多训起来会有梯度问题，所以Darknet-19也就19层，因此得益于ResNet的residual结构，训深层网络难度大大减小，因此这里可以将网络做到53层，精度提升比较明显）。
　　Darknet-53只是特征提取层，源码中只使用了pooling层前面的卷积层来提取特征，因此multi-scale的特征融合和预测支路并没有在该网络结构中体现，具体信息可以看源码：https://github.com/pjreddie/darknet/blob/master/cfg/yolov3.cfg。预测支路采用的也是全卷积的结构，其中最后一个卷积层的卷积核个数是255，是针对COCO数据集的80类：3X(80+4+1)=255，3表示一个grid cell包含3个bounding box，4表示框的4个坐标信息，1表示objectness score。模型训练方面还是采用原来YOLO v2中的multi-scale training。

<div align=center>  
![](http://ow7va355d.bkt.clouddn.com/yolo-27.png)
</div>

# 三、参考文献

1. [YOLO v3算法笔记](https://blog.csdn.net/u014380165/article/details/80202337)
2. [yolo-v2论文](https://pjreddie.com/media/files/papers/YOLOv3.pdf)
3. [从零开始PyTorch项目：YOLO v3目标检测实现](https://juejin.im/entry/5adddff86fb9a07aa0479e9f)











