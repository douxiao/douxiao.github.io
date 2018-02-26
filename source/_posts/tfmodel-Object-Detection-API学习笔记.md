---
title: tfmodel_Object_Detection_API学习笔记
date: 2017-12-19 22:50:51
tags:
- tfmodel
- tensorflow
categories: 工作
---
<font size="5" color="lightseagreen"> 这篇博客是总结自己在使用tensorflow models里面的物体检测的API的方法以及调试中遇到的错误及心得。</font>

<!--more-->

# [](#一、安装 "一、安装")**一、安装**

## [](#1、dependencies "1、dependencies")1、dependencies
<figure class="highlight python"><table><tr><td class="gutter"><pre><div class="line">1</div><div class="line">2</div><div class="line">3</div><div class="line">4</div><div class="line">5</div><div class="line">6</div><div class="line">7</div></pre></td><td class="code"><pre><div class="line">Protobuf <span class="number">2.6</span></div><div class="line">Pillow <span class="number">1.0</span></div><div class="line">lxml</div><div class="line">tf Slim (which <span class="keyword">is</span> included <span class="keyword">in</span> the <span class="string">"tensorflow/models/research/"</span> checkout)</div><div class="line">Jupyter notebook</div><div class="line">Matplotlib</div><div class="line">Tensorflow</div></pre></td></tr></table></figure>

## [](#2、install-tensorflow "2、install tensorflow")2、install tensorflow

 For detailed steps to install Tensorflow, follow the [Tensorflow installation instructions](https://www.tensorflow.org/install/)

## [](#3、install-denpendencies "3、install denpendencies")3、install denpendencies

  The remaining libraries can be installed on Ubuntu 16.04 using via apt-get:

<figure class="highlight python"><table><tr><td class="gutter"><pre><div class="line">1</div><div class="line">2</div><div class="line">3</div></pre></td><td class="code"><pre><div class="line">sudo apt-get install protobuf-compiler python-pil python-lxml</div><div class="line">sudo pip install jupyter</div><div class="line">sudo pip install matplotlib</div></pre></td></tr></table></figure>

Alternatively, users can install dependencies using pip:

<figure class="highlight python"><table><tr><td class="gutter"><pre><div class="line">1</div><div class="line">2</div><div class="line">3</div><div class="line">4</div></pre></td><td class="code"><pre><div class="line">sudo pip install pillow</div><div class="line">sudo pip install lxml</div><div class="line">sudo pip install jupyter</div><div class="line">sudo pip install matplotlib</div></pre></td></tr></table></figure>

## [](#4、protobuf-compilation "4、protobuf compilation")4、protobuf compilation

 Tensorflow Object Detection API使用Protobufs来配置模型和训练参数。在可以使用框架之前，必须编译Protobuf库。这应该通过从tensorflow/ models/research/目录运行以下命令来完成:
<figure class="highlight python"><table><tr><td class="gutter"><pre><div class="line">1</div><div class="line">2</div></pre></td><td class="code"><pre><div class="line"><span class="comment"># From tensorflow/models/research/</span></div><div class="line">protoc object_detection/protos/*.proto --python_out=.</div></pre></td></tr></table></figure>

## [](#6、Testing-the-Installation "6、Testing the Installation")6、Testing the Installation

 You can test that you have correctly installed the Tensorflow Object Detection
API by running the following command:

`python object_detection/builders/model_builder_test.py`

# [](#二、实际使用经验 "二、实际使用经验")**二、实际使用经验**

## [](#1、安装tensorflow "1、安装tensorflow")1、安装tensorflow

   自己采用的是virtualenv安装虚拟python环境的tensorflowpy2的models 放在tensorflowpy2/lib/python2.7/site-ackages/tensorflow/models路径下。
   用pycharm新建工程路径为models/research/object_detection。并使用该虚拟的python27环境。执行上面一系列的编译环境。
   然后自己执行，object_detection文件下面的object_detection_tutorial.ipynb文件,这里会下载一些权重参数可能会比较慢。
   在实行imports这个框的时候，需要把1.4.0版本改成自己对应的版本。

## [](#2、遇到的问题 "2、遇到的问题")2、遇到的问题

IOError: (‘http error’, 302, ‘Found’, <httplib.httpmessage instance="" at="" 0x7f6ed7dc40e0="">)</httplib.httpmessage>

这个错误的解决办法是:需要重新编译proto。

如图![](http://ow7va355d.bkt.clouddn.com/Screenshot%20from%202017-12-20%2010-39-11.png)

<font color="red" size="10"> 解决No module named ‘object_detection’问题</font>

这个问题困扰了自己很久今天终于解决了，因为自己的tensorflow是安装在virualenv虚拟环境下的，开始自己以为虚拟环境下的tensorflow无法加载在系统环境变量下安装的modules，但是觉得应该有解决办法，下面是自己的探索之路:

<font color="purple">** 探索一、**虚拟环境如何加载系统环境:</font>
实测默认情况下虚拟环境不会依赖系统环境的global site-packages。比如系统环境里安装了MySQLdb模块，在虚拟环境里import MySQLdb会提示ImportError。如果想依赖系统环境的第三方软件包，可以使用参数–system-site-packages。

`virtualenv --system-site-packages [虚拟环境名称]`

[参考链接:virtualenv虚拟环境](https://snailvfx.github.io/2016/05/11/virtualenv/)

 <font color="purple">** 探索二、**如何把models中的库添加到系统的python环境中:</font>

<figure class="highlight python"><table><tr><td class="gutter"><pre><div class="line">1</div><div class="line">2</div></pre></td><td class="code"><pre><div class="line">  <span class="comment"># From tensorflow/models/research/</span></div><div class="line">export PYTHONPATH=$PYTHONPATH:`pwd`:`pwd`/slim</div></pre></td></tr></table></figure>

 pwd是指代当前目录的环境，需要替换过成自己的，比如自己在 ~/.bashrcw文件中这样声明的
 <figure class="highlight python"><table><tr><td class="gutter"><pre><div class="line">1</div></pre></td><td class="code"><pre><div class="line">export PYTHONPATH=$PYTHONPATH:/data/tensorflowModels/models/research:/data/tensorflowModels/models/research/slim</div></pre></td></tr></table></figure>

 接下来还要编译Protobuf
 <figure class="highlight python"><table><tr><td class="gutter"><pre><div class="line">1</div><div class="line">2</div></pre></td><td class="code"><pre><div class="line"> <span class="comment"># From tensorflow/models/research/</span></div><div class="line">protoc object_detection/protos/*.proto --python_out=.</div></pre></td></tr></table></figure>

然后，自己重新安装了tensorflow加载系统的环境变量还是出错。

 [参考链接:安装modles](https://github.com/tensorflow/models/blob/4f32535fe7040bb1e429ad0e3c948a492a89482d/research/object_detection/g3doc/installation.md)

  <font color="purple">** 探索三、**安装slim执行以下命令:</font>

 在slim目录下执行下面命令

 <figure class="highlight python"><table><tr><td class="gutter"><pre><div class="line">1</div><div class="line">2</div></pre></td><td class="code"><pre><div class="line">python setup.py build</div><div class="line">python setup.py install</div></pre></td></tr></table></figure>

 [参考：安装slim](https://github.com/tensorflow/models/issues/2031)

接下来就可以使用了。

[参考博客:TensorFlow之物体检测](http://rensanning.iteye.com/blog/2381885) 