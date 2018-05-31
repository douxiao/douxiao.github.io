---
title: 深度学习——YOLO-v1论文阅读笔记
date: 2018-05-30 10:18:04
tags:
- yolo
- object-detection
categories: 论文阅读
---
<font size="5" color="lightseagreen"> 很久之前就读过YOLO的论文，因为最近YOLO-v3已经发布，有必要系统的做一下总结，这篇博客是总结YOLO-v1，在15年11份发布的版本。
 </font>
<!--more-->

# 一、背景介绍

　　YOLO将第一次实现将物体检测作为回归问题求解。基于一个单独的end-to-end网络，完成从原始图像的输入，到**物体位置和类别及相应的置信概率**的输出，在目标检测网络中属于one-stage 方法。

　　下图是yolo和基于R-CNN系列的目标检测的对比：
<div align=center>
![](http://ow7va355d.bkt.clouddn.com/yolo-vs-rcnn.jpg)

图1
</div>

　　下图是YOLO和其他网络的性能对比图：

<div align=center>
![](http://ow7va355d.bkt.clouddn.com/yolo-19.png)
</div>


# 二、核心算法


## 2.1 网络结构


　　YOLO网络结构借鉴了GoogLeNet分类网络结构，但未使用Inception module，而是使用的是1X1卷积层（1X1是为了跨通道信息整合，同时可以认为应用了一个全卷积神经网络）同时还使用了3X3卷积层简单替代。


　　**下图**：卷积层用来提取图像特征，全连接层用来预测图像位置和类别概率值。

<div align = center>
　![](http://ow7va355d.bkt.clouddn.com/yolo1_network_arch.png)
图2
　![](http://ow7va355d.bkt.clouddn.com/yolo-1.png)
图3
</div>

## 2.2网络实现图解


图解YOLO，如下图所示：

<div align=center>

　![](http://ow7va355d.bkt.clouddn.com/yolo-4.png)
![](http://ow7va355d.bkt.clouddn.com/yolo-5.png)
![](http://ow7va355d.bkt.clouddn.com/yolo-6.png)
![](http://ow7va355d.bkt.clouddn.com/yolo-7.png)
![](http://ow7va355d.bkt.clouddn.com/yolo-8.png)
![](http://ow7va355d.bkt.clouddn.com/yolo-9.png)
![](http://ow7va355d.bkt.clouddn.com/yolo-10.png)
![](http://ow7va355d.bkt.clouddn.com/yolo-11.png)
![](http://ow7va355d.bkt.clouddn.com/yolo-12.png)
![](http://ow7va355d.bkt.clouddn.com/yolo-13.png)

图4
</div>

## 2.3损失函数

　　YOLO使用均方和误差作为loss函数来优化模型参数，即网络输出的SxSx(Bx5 + C)维向量与真实图像的对应SxSx(Bx5 + C)维向量的均方和误差。如下式所示。其中，coordError、iouError和classError分别代表**预测数据与标定数据之间的坐标误差**、**IOU误差**和**分类误差**。

![](http://ow7va355d.bkt.clouddn.com/yolo-14.png)

　　YOLO对上式loss的计算进行了如下修正。

　　[1]位置相关误差（坐标、IOU）与分类误差对网络loss的贡献值是不同的，因此YOLO在计算loss时，使用λ{coord} = 5修正coordError。

　　[2]在计算IOU误差时，包含物体的格子与不包含物体的格子，二者的IOU误差对网络loss的贡献值是不同的。若采用相同的权值，那么不包含物体的格子的confidence值近似为0，变相放大了包含物体的格子的confidence误差在计算网络参数梯度时的影响。为解决这个问题，YOLO 使用λ{noobj} =0.5修正iouError。（注此处的‘包含’是指存在一个物体，它的中心坐标落入到格子内）。

　　[3]对于相等的误差值，大物体误差对检测的影响应小于小物体误差对检测的影响。这是因为，相同的位置偏差占大物体的比例远小于同等偏差占小物体的比例。YOLO将物体大小的信息项（w和h）进行求平方根来改进这个问题。（注：这个方法并不能完全解决这个问题）。

综上，YOLO在训练过程中Loss计算如下式所示：

loss计算如下式：

![](http://ow7va355d.bkt.clouddn.com/yolo-15.jpg)

![](http://ow7va355d.bkt.clouddn.com/yolo-16.png)


## 2.4 训练
　　YOLO模型训练分为两步：
　　1）预训练。使用ImageNet
1000类数据训练YOLO网络的前20个卷积层+1个average池化层+1个全连接层。训练图像分辨率resize到224x224。

　　2）用步骤1）得到的前20个卷积层网络参数来初始化YOLO模型前20个卷积层的网络参数，然后用VOC 20类标注数据进行YOLO模型训练。为提高图像精度，在训练检测模型时，将输入图像分辨率resize到448x448。

# 三、总结
　　综上，YOLO具有以下的几个优点：
　　　1.**速度快**，yolo将物体检测作为回归问题求解，整个网络简单，在titan x GPU上，在保证检测准确率的前提下（63.4% mAP，VOC 2007 test set），可以达到45fps的检测速度。
　　　2.**背景误检率低**，YOLO在训练和推理过程中能‘看到’整张图像的整体信息，而基于region proposal的物体检测方法（如rcnn/fast rcnn），在检测过程中，只‘看到’候选框内的局部图像信息。因此，**若当图像背景（非物体）中的部分数据被包含在候选框中送入检测网络进行检测时，容易被误检测成物体。**测试证明，YOLO对于背景图像的误检率低于fast rcnn误检率的一半。


　　缺点是：
　　1. 识别物体位置精准性差。
　　2. 召回率低。

# 四、参考文献

1. [YOLO详解](https://zhuanlan.zhihu.com/p/25236464)
2. [YOLO PPT](https://docs.google.com/presentation/d/1aeRvtKG21KHdD5lg6Hgyhx5rPq_ZOsGjG5rJ1HP7BbA/pub?start=false&loop=false&delayms=3000&slide=id.p)
3. [YOLO论文](https://arxiv.org/abs/1506.02640)
4. [github](https://github.com/thtrieu/darkflow)



























