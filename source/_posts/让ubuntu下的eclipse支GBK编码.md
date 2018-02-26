---
title: 让ubuntu下的eclipse支GBK编码
date: 2017-10-13 13:51:29
tags:
- GBK编码
- eclipse
categories: 工作
---

　<font size="5" color="lightseagreen">  今天，把windows下的工程导入到了Linux下eclipse中，由于以前的工程代码，都是GBK编码的，而Ubuntu默认是不支持GBK编码的。所以，首先我们要先让Ubuntu支持GBK，方法如下:</font>
<a id="more"></a>

1、修改/var/lib/locales/supported.d的权限
  `sudo chmod -R 777 /var/lib/locales/supported.d`
2、打开/var/lib/locales/supported.d/local文件,在文件中添加
（打开文件命令为`gedit /var/lib/locales/supported.d/local`）
     zh_CN.GBK GBK
     zh_CN.GB2312 GB2312
3、在终端输入
  `sudo dpkg-reconfigure --force locales`
4、设置eclipse:Windows-&gt;Preferences-&gt;General-&gt;Workspace在Text file encoding下选择other在后面的文本框中输入GBK点击应用即可 ,如果没有GBK的选项， 没关系，直接输入GBK三个字母，Apply，GBK编码的中文，已经不是乱码了。。