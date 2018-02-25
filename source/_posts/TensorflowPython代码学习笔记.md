---
title: TensorflowPython代码学习笔记
date: 2017-11-04 14:32:48
tags:
- tensorflow
- python
categories: 工作
---
　　最近在学习Tesorflow机器学习实战这本书，之前一直在学习深度学习相关的理论的知识，如果自己动手写代码，真的无从下手，这本书虽然理论知识很少，但是在tensorflow编程实战，是一个很不错的书，这篇博客是总结在写代码时自己的遇到的不会的地方，也可以说学习tensoflow和python编程笔记。
<!--more-->

1、`train_indices = np.random.choice(len(x_vals), round(0.8*len(x_vals)), replace=False)`
　　round(0.8*len(x_vals))该方法返回 0.8*len(x_vals)的小数点四舍五入到n个数字
   函数原型numpy.random.choice(a, size=None, replace=True, p=None)
-    该函数的作用是从给定的一维数组中生成随机数
-    a为一维数组类似数据或整数；size为数组维度；p为数组中数据出现的概率
-    a为整数时，对应的一位数组为np.arange(a)

```python
x_val = np.random.choice(5, 3, replace=False)
# 当replace为False时，生成随机数不能有重复的数值
print(x_val)
[1 0 4]
y_val = np.random.choice(5,size=(3,2))
print(y_val)
[[1, 0],
 [4, 2],
 [3, 3]]
demo_list = ['lenovo', 'sansumg','moto','xiaomi', 'iphone']
np.random.choice(demo_list,size=(3,3))
array([['moto', 'iphone', 'xiaomi'],
       ['lenovo', 'xiaomi', 'xiaomi'],
       ['xiaomi', 'lenovo', 'iphone']],
      dtype='<U7')
# 参数p的长度与参数a的长度需要一致； 
# 参数p为概率，p里的数据之和应为1
demo_list = ['lenovo', 'sansumg','moto','xiaomi', 'iphone']
np.random.choice(demo_list,size=(3,3), p=[0.1,0.6,0.1,0.1,0.1])
array([['sansumg', 'sansumg', 'sansumg'],
       ['sansumg', 'sansumg', 'sansumg'],
       ['sansumg', 'xiaomi', 'iphone']],
      dtype='<U7')
```
２、 numpy.random.seed()

- np.random.seed()的作用：使得随机数据可预测。
- 当我们设置相同的seed，每次生成的随机数相同。如果不设置seed，则每次会生成不同的随机数。

```python
np.random.seed(0)
np.random.rand(5)
array([ 0.5488135 ,  0.71518937,  0.60276338,  0.54488318,  0.4236548 ])
np.random.seed(1676)
np.random.rand(5)
array([ 0.39983389,  0.29426895,  0.89541728,  0.71807369,  0.3531823 ])
np.random.seed(1676)
np.random.rand(5)
array([ 0.39983389,  0.29426895,  0.89541728,  0.71807369,  0.3531823 ])
```












