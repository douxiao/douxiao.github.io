---
title: 深度学习——CNN卷积神经网络
date: 2018-05-10 19:24:41
tags:
- CNN
- DeepLearning

categories: 工作
---
<font size="5" color="lightseagreen">   卷积神经网络的生动总结，加深理解！
 </font>
<!--more-->

# 一、前言
　　卷积神经网络的产生是为了<font color=red >** 解决深层神经网络参数多训练难的问题，并获得更好的分类效果。** </font>  深度学习出现之前，传统的神经网络层数变多时，反向传播将会变得非常困难，并且对一些高维的输入往往需要大量的参数，因此复杂的神经网络的训练变得非常困难。同时**由于全连接网络是提取前一层的全部信息，整体的特征拟合能力较低，且易于拟合于局部样本。**

　　**卷积神经网络的实现来源于对视觉神经系统的研究**，科学家发现，**人对事物的观察是通过对不同特征捕获的综合**，视觉神经系统中有专门负责不同特征感知的视觉元。从对视觉神经系统的研究出发，神经网络的研究者提出用一个神经元每次观察输入的部分特征（比如图片的一小块），然后通过逐步移动的方法观察整个输入的方法，然后**用多个这种神经元提取输入的输入的不同特征，最后在通过一个全连接网络对这些特征进行整合，最终达成分类效果**。下图形象的描述了这种特征的提取。
<div align=center>
![](http://ow7va355d.bkt.clouddn.com/%E5%85%A8%E8%BF%9E%E6%8E%A5%E4%B8%8E%E7%A5%9E%E7%BB%8F%E5%85%83.jpg)
</div>

　　通过多个神经元代替一个全连接层，参数量可以大大的减少，且由于图像存在大量的相同的基本特征，所以卷积神经网络在计算机视觉领域取得了很好的效果。
  
# 二、简介

**卷积神经网络就是由卷积层、池化层、全连接层构成的具有局部感知和权值共享能力的深层神经网络**
<div align=center>
![](http://ow7va355d.bkt.clouddn.com/CCN1)
</div> 
 卷积神经网络的特点：
 1）**局部连接** 。 可以提取数据的局部特征。
 2）**权值共享** 。 大大的降低了训练的难度。
 3）**池化操作** 。 数据降维的同时，保留有用的信息。
 4）**全连接层** 。 对提取的特征进行组织综合，输出识别物体的分类情况。
<div align=center>
![](http://ow7va355d.bkt.clouddn.com/CNN2.png)
</div> 


# 三、重要概念

## 3.1卷积
   卷积的本质就是加权叠加，最后通过滑动窗口（filter）观察整个输入，输出一个feature map。
###    一维卷积：
<div align=center>
![](http://img.my.csdn.net/uploads/201301/09/1357719312_6678.png)
</div>
   图中的M向量为卷积核，N向量为输入，P向量为输出。其中P[2] = N[0] * M[0] + … + N[4] * M[4]。

### 二维卷积：（图像）  
<div align=center>
![](https://img-blog.csdn.net/20170503110450187?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0NTY1OTM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
</div>


卷积层中卷积核的使用，一般如上图所示。卷积层是卷积核在上一级输入层上通过逐一滑动窗口计算而得，卷积核中的每一个参数都相当于传统神经网络中的权值参数，与对应的局部像素相连接，将卷积核的各个参数与对应的局部像素值相乘之和，（通常还要再加上一个偏置参数），得到卷积层上的结果（即feature map）。
<div align=center>
![](https://img-blog.csdn.net/20160702215705128)
</div>

### 多通道多卷积核
<div align=center>
 ![](https://img-blog.csdn.net/20160707204048899)
</div>



1) input: 7*7*3 RGB)
2) fiters: 3*3  strides: 2
3) output: 2 feature maps  hidden layers:2

说明： **卷积核的权重矩阵就是训练时要学习的参数，就是要提取的特征，神经网络再根据卷积提取的特征去观察输入。**
<div align=center>
 ![](http://upload-images.jianshu.io/upload_images/145616-cdf0a3911ba67b0f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
</div>


# 3.2激活函数

   卷积神经网络中，最常用的激活函数是relu : f(x)= max(0, x)
<div align=center>
![](http://upload-images.jianshu.io/upload_images/2256672-0ac9923bebd3c9dd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/640)
</div>



**Relu函数作为激活函数，有下面几大优势：**

1)  **速度快 **和sigmoid函数需要计算指数和倒数相比，relu函数其实就是一个max(0,x)，计算代价小很多。
2) **减轻梯度消失问题** 回顾计算梯度的公式∇=σ′δx。其中，σ′是sigmoid函数的导数。在使用反向传播算法进行梯度计算时，每经过一层sigmoid神经元，梯度就要乘上一个σ′。从下图可以看出，σ′函数最大值是1/4。因此，乘一个会导致梯度越来越小，这对于深层网络的训练是个很大的问题。而relu函数的导数是1，不会导致梯度变小。当然，激活函数仅仅是导致梯度减小的一个因素，但无论如何在这方面relu的表现强于sigmoid。使用relu激活函数可以让你训练更深的网络。
3) **稀疏性** 通过对大脑的研究发现，大脑在工作的时候只有大约5%的神经元是激活的，而采用sigmoid激活函数的人工神经网络，其激活率大约是50%。有论文声称人工神经网络在15%-30%的激活率时是比较理想的。因为relu函数在输入小于0时是完全不激活的，因此可以获得一个更低的激活率。
<div align=center>
![](http://upload-images.jianshu.io/upload_images/145616-2c9ee822f6daa1e1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
</div>


# 3.3池化层
<div align=center>
![](https://img-blog.csdn.net/20160703121026432)
</div>
池化层里我们用的maxpooling，将主要特征保留，舍去多余无用特征,这样就可以实现信息压缩，比如下图所示 
<div align=center>
![](http://upload-images.jianshu.io/upload_images/145616-88c19d8616068d06.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
</div>

　　经过多轮的卷积和池化，将得到最终的特征表示

<div align=center>
![](http://upload-images.jianshu.io/upload_images/145616-25c57dafffd49ce8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
</div>


# 3.4 FC层

　　全连接层(Fully connected layers ,FC)在整个卷积神经网络中起到“分类器”的作用。如果说卷积层、池化层和激活函数等操作将原始数据映射到隐层特征空间的话，全连接层则起到将**分布式特征表示**映射到样本标记空间的作用。
　　**在实际使用中，全连接层可由卷积操作实现：对前层是全连接的全连接层可以转化为卷积核为1x1的卷积；而前层是卷积层的全连接层可以转化为卷积核为hxw的全局卷积，h和w分别为前层卷积结果的高和宽（注1）。**
　　++目前由于全连接层参数冗余（仅全连接层参数就可占整个网络参数80%左右），近期一些性能优异的网络模型如ResNet和GoogLeNet等均用全局平均池化（global average pooling，GAP）取代FC来融合学到的深度特征，最后仍用softmax等损失函数作为网络目标函数来指导学习过程。++需要指出的是，用GAP替代FC的网络通常有较好的预测性能。

　　**注1: **有关卷积操作“实现”全连接层，有必要多啰嗦几句。以VGG-16为例，对224x224x3的输入，最后一层卷积可得输出为7x7x512，如后层是一层含4096个神经元的FC，则可用卷积核为7x7x512x4096的全局卷积来实现这一全连接运算过程，其中该卷积核参数如下：“filter size = 7, padding = 0, stride = 1, D_in = 512, D_out = 4096”经过此卷积操作后可得输出为1x1x4096。如需再次叠加一个2048的FC，则可设定参数为“filter size = 1, padding = 0, stride = 1, D_in = 4096, D_out = 2048”的卷积层操作。

　　最后用全连接对特征进行拟合：
<div align=center>
 ![](http://upload-images.jianshu.io/upload_images/145616-e2e9a7c59444b138.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
</div>

　　并输出不同分类的概率
<div align=center>
  ![](http://upload-images.jianshu.io/upload_images/145616-e507135cb66e02b2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
</div>


　　**问题：**怎样把一个3*3*5的输出，转化成1*4096形式？

<div align=center>
  ![](https://pic3.zhimg.com/80/v2-4d1ed82851e96dd58620f451d8c1e98e_hd.jpg)

</div>

　　可以理解为在中间做一个卷积：

<div align=center>
  ![](https://pic4.zhimg.com/80/v2-677c85adc52245a0c3bd6c766e057da8_hd.jpg)
</div>

　　从上图我们可以看出，我们用一个3x3x5的filter 去卷积激活函数的输出，得到的结果就是一个fully connected layer 的一个神经元的输出，这个输出就是一个值因为我们有4096个神经元我们实际就是用一个3x3x5x4096的卷积层去卷积激活函数的输出。


# 3.5 卷积神经网络的可视化
<div align=center>
 ![](https://img-blog.csdn.net/20160702205047459)
</div>













