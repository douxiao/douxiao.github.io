---
title: Python-Numpy学习
date: 2017-09-23 10:51:53
tags:
- python
- numpy
categories: 工作
---
<font size=5>Python的科学计算包—Numpy</font>

　　numpy是一个第三方的python包，用于科学计算。这个库的前身是1995年开始开发的一个用于数组运算的库。经过了长时间的发展，基本上成了绝大部分python科学计算的基础包，同时提供python接口的深度学习框架。
<!--more-->

一、基本类型（array）
　　array,也就是数组，是numpy中最基础的数据结构，最关键的属性是维度和元素类型，在numpy中，可以非常方便的创建各种不同类型的多维数组，并且执行一些基本的操作。
``` python
import numpy as np
a = [1, 2, 3, 6]
b = np.array(a)         # array([[1, 2, 3], [4, 5, 6]])
type(b)                 # <class 'numpy.ndarray'>
b.shape                 # (4, )
b.argmax()              # 3
b.max()                 # 4
b.mean()                # 3.0

c = [[1, 2], [3, 4]]    # 二维列表
d = np.array(c)         # 二维numpy数组
d.shape                 # (2, 2)
d.size                  # 4
d.max(axis=0)           # 对于二维数组，代表每一列的最大值[3 4] 
d.max(axis=1)           # 对于二维数组，代表每一行的最大值[2 4]
d.mean(axis=0)          # 对于二维数组，代表每一列的均值[2. 3.]
d.flatten()             # 展开一个numpy数组为一维数组[1 2 3 4]
np.ravel(c)             # 展开一个可以解析的结构为1维的数组[1 2 3 4]

# 3x3的浮点型二维数组，并且初始化所有的元素值为1
e = np.ones((3, 3), dtype=np.float32)  

# 创建一个一维数组，元素值是把3重复4次 [3 3 3 3]
f = np.repeat(3, 4)

# 2x2x3的无符号8位整形的3维数组，并且初始化所有的值为0
g = np.zeros((2, 3, 3), dtype=np.uint8, )
g.shape                      # (2,3,3)
h = g.astype(np.float)       # 转换为浮点型数据
l = np.arange(10)            # [0 1 2 3 4 5 6 7 8 9]
m = np.linspace(0, 6, 5)     # 等差数列，0到6之间去五个值 [ 0.   1.5  3.   4.5  6. ]
p = np.array(
    [[1, 2, 3, 4],
     [5, 6, 7, 8]]
)
np.save('p.npy', b)          # 保存到文件
q = np.load('p.npy')         # 从文件中读取

```