---
title: Nvidia-TX1刷机教程
date: 2017-10-26 08:30:16
tags:
- Nvidia-Tx1
- 嵌入式GPU 
categories: 工作
---
　　这篇博客是总结这几天Nvidia Tx1开发板的刷机血泪史，总结遇到的一些问题及解决办法，之所以艰难是因为实验室的网络太差，还有最近十九大的召开，国内所有的vpn都被禁了，下载某些安装包是很困难的，没法google，百度有些问题的解决办法是无法找到的,不过也参考了一些前辈的教程。不过值得庆幸的事，最终还是刷机成功了，写一篇博文来纪念一下。
<!--more-->

# 一、开箱检查，并安装系统

　　开发板连接好AC电源线，**使用HDMI线连接显示器**，自己在这里踩坑了，因为开始用的hdmi转vga，显示器怎么都不显示，用HDMI直连显示器就可以了，开始我的板子会进入命令行界面，开始还没有安装系统，所以没有桌面，需要运行下安装。
　　然后登录系统，用户名和密码都是”ubuntu“，登陆后，系统会提示安装驱动以显示正常的图像界面。按照要求进行操作，3步以后驱动就安好了，然后sudo reboot重启系统，就可以进入ubuntu 14.04系统界面。终端输入sudo lshw就可以查看系统硬件信息，基本可以确定开发板的完好。
　　<font color=red >注意:开始这里必须用hdmi线直连显示器。</font>
  

# 二、刷机准备，下载安装JetPack3.1

　  开发板安装软件的一个方式是使用JetPack,Jetson TX1出厂时默认的系统以及附加包都比较老旧,没有安装cuda,nvcc,opencv等，部署最新的开发包可以充分利用硬件性能，有利于我们进行深度学习开发。Jetpack3.1是Nvidia提供的最新开发包，包含64bit的Ubuntu16.04操作系统，CUDA 8.0，cuDNN 6.0等。类似于刷安卓手机，我们需要在一台装有ubuntu14.04或16.04的电脑上为开发板更新固件。
## 第一步：
　　　下载JetPack3.1，需要注册英伟达开发者账号，[官网链接](https://developer.nvidia.com/embedded/jetpack)，[JetPack Download  center](https://developer.nvidia.com/embedded/downloads)。
   ![](http://ow7va355d.bkt.clouddn.com/4.png)
## 第二步：
　　　安装JetPack3.1，因为安装需要大概10G左右的空间，所以我没有放在系统的那个盘安装。加入我放在data盘中。
   ```python
   cd /data
   mkdir NvidiaTx1 //然后把下载的文件拷贝到该文件夹的目录下面。
   cd NvidiaTx1
   ./JetPack-L4T-3.1-linux-x64.run  
```
　　这里会进入安装界面，点next,选择TX1,会进入到下面那个界面，因为实验室网络经常抽风，经常会在下面那个界面卡死报错，解决办法重复进几次，自己有一次尝试打开vpn居然就能用，还有把4G网共享给电脑，总之试了很多办法。如果运气好的话，会很顺利完成的。
  
  ![](http://ow7va355d.bkt.clouddn.com/Jetpack_install3.png)
  
　　应该是下面的这个效果，当然自己写教程前已经安装好了，所以在主机部分显示的是no action,正常的应该显示install版本，接下来的下载将是一个漫长的，不断失败的过程，我也搞不懂为什么反正有种绝望想放弃的感觉。
  
  ![](http://ow7va355d.bkt.clouddn.com/Jetpack_install4.png)
　　
　　好了点了next,就去忙别的吧，当然下载错误还是那些办法，重新开始，换网。。。。。。
　　经过了漫长的等待下载完成。
  
## 第三步：

　　安装：全部下载完后，开始了每一项的安装，此时可能会报出cuda安装失败的错误，此时查看日志文件，多半能找到答案。我的做法是打开终端，运行`sudo apt-get -f install`命令，补全依赖项，然后就可以顺利安装。
  
# 三、开始刷机  

　　刚才开发板所需组件全部下载并安装后，就可以准备刷机了。
##   第一步：配置网络

　　开发板刷机过程中需要全程联网，那么官方推荐的做法就是把电脑与开发板用网线连在同一个路由器下，至于无线连接行不行我没试过，不过为了保证稳定，建议使用网线。那么在弹出的network layout配置中选择路由连接；在network interface中选择以太网端口，不认识的话就用默认选项。 
  ![](http://ow7va355d.bkt.clouddn.com/10.png)
  ![](http://ow7va355d.bkt.clouddn.com/11.png)
  ![](http://ow7va355d.bkt.clouddn.com/12.png)
  ![](http://ow7va355d.bkt.clouddn.com/13.png)
  
## 第二步、开发板连接到电脑，开始刷机

　1.断开电源，保证开发板处于断电关机状态。
　2.用网线连到路由器上，也可插上鼠标键盘。
　3.用Micro USB线把开发板连到电脑上（类似于安卓手机连电脑）接通AC电　源，按下power键，开机。 
　4.长按rec键不松开，然后点按一下reset键(这里会看到灯闪一下)，过2s以后，才松开rec键，此时开发板处于强制恢复模式。

　　完成以上步骤后，我们还要检查开发板有没有和电脑正确连接，终端输入lsusb 命令，可以看到一些列表，只要发现其中有Nvidia Corp就说明连接正确。
　　以上步骤确认无误后，在post installation界面中敲一下enter，就开始了刷机过程。  
　　<font color=red> **注意在这里自己遇到一个错误。Making   　　　system.img... 
　　　rm: cannot remove 'mnt': Device or resource busy
　　　clearing ext4 mount point failed.**</font>
　　这里需要手动清楚 /mnt文件及目录。[解决办法链接](https://devtalk.nvidia.com/default/topic/1021063/jetson-tx2/failed-to-flash-device/post/5197174/)。
  
　　好了下面可以进行正常刷机了。
  
  ![](http://ow7va355d.bkt.clouddn.com/14.png)
　　
　　刷机过程中，会出现提示确认GUI桌面是否安装好，此时用HDMI线缆连到显示器上，如果显示ubuntu桌面，说明系统安装好了，按照提示完成后续安装，这将是一个持续几十分钟的过程。完全安好后，退出Jetpack软件即可。  
**　　自己这里遇到的问题是检测不到tx1的IP。只能退出，下面讲解怎样使用JetPack仅安装组件。，如果这个过程很慢而且出错的话那只能手动安装cuda ,openv 等组件了。**
  
# 四、JetPack仅安装组件  
　　这就得提到Jetpack的另一个特性：可以不必刷机，单独为Jetson设备安装任何组件。方法其实很简单： 
  
  　![](http://ow7va355d.bkt.clouddn.com/Jetpack_install4.png)
   
   类似上图，把FileSystem and OS、Drivers、Flash OS image to Target这些关于系统的组件通通置为no action，然后选择需要补充安装的组件，注意它们的依赖关系。选择完毕就点next，会出现如下界面，需要填上Tx1的IP： 
   
 ![](http://ow7va355d.bkt.clouddn.com/IPinstall.png)
   
   就可以开始安装了。这里使用的是SSH远程服务，根本不用数据线，等待一会就安装好了，注意安装过程中尽量不要操作开发板。
  

  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  