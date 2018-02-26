---
title: surface及surfaceview使用总结
date: 2017-10-30 09:20:21
tags:
- android
categories: 工作
---
  

　<font size="5" color="lightseagreen">这篇博客是关于android中的surface、surfaceView以及hoder的使用总结。
 </font>
<!--more-->

# [](#一、什么是Surface "一、什么是Surface")一、什么是Surface

　　简单的说Surface对应了一块屏幕缓冲区，每个window对应一个Surface，任何View都要画在Surface的Canvas上（后面有原因解释）。传统的view共享一块屏幕缓冲区，所有的绘制必须在UI线程中进行。
　　<font color="blue">Surface中的Canvas成员，是专门用于供程序员画图的场所，就像黑板一样；其中的原始缓冲区是用来保存数据的地方；Surface本身的作用类似一个句柄，得到了这个句柄就可以得到其中的Canvas、原始缓冲区以及其它方面的内容。</font>

# [](#二、什么是SurfaceView "二、什么是SurfaceView")二、什么是SurfaceView

　　说SurfaceView是一个View也许不够严谨，然而从定义中pubilc class　SurfaceView extends View{…..}显示SurfaceView确实是派生自View，但是SurfaceView却有自己的Surface，请看SurfaceView的源码。
　　SurfaceView就是展示Surface中数据的地方，同时可以认为SurfaceView是用来控制Surface中View的位置和尺寸的。

# [](#三、什么是SufaceHolder "三、什么是SufaceHolder")三、什么是SufaceHolder

　　SurfaceHolder是一个接口，其作用就像一个关于Surface的监听器，提供访问和控制SurfaceView内嵌的Surface 相关的方法。它通过三个回调方法，让我们可以感知到Surface的创建、销毁或者改变。

<font color="red">　　 总结：从设计模式的高度来看，Surface、SurfaceView和SurfaceHolder实质上就是广为人知的MVC，即Model-View-Controller。Model就是模型的意思，或者说是数据模型，或者更简单地说就是数据，也就是这里的Surface；View即视图，代表用户交互界面，也就是这里的SurfaceView；SurfaceHolder很明显可以理解为MVC中的Controller（控制器）。</font>

# [](#四、什么是SurfaceHolder-Callback "四、什么是SurfaceHolder.Callback")四、什么是SurfaceHolder.Callback

　　SurfaceHolder.Callback主要是当底层的Surface被创建、销毁或者改变时提供回调通知，由于绘制必须在Surface被创建后才能进行，因此SurfaceHolder.Callback中的surfaceCreated 和surfaceDestroyed 就成了绘图处理代码的边界。