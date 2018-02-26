---
title: Ubuntu软件创建图标
date: 2017-12-20 22:50:51
tags:
- desktop
- ubuntu
categories: 工作
---

　<font size="5" color="lightseagreen">  自己习惯了用ubuntu，特别是在安装了mac主题桌面后有一个非常棒的UI，但是美中不足的是安装一些软件后系统不能像windows下面那样自动创建图标，最近发现了一个特别有用的教程,用来创建Ubuntu图标。</font>

<!--more-->


举例。给自己的系统创建pycharm的图标
```python
cd /data/opt/pycharm
touch pycharm.desktop
gedit pycharm.desktop

```
在打开的文件中填写以下代码
```python
[Desktop Entry]
Name=Pycharm
Type=Application
Icon=/data/opt/pycharm/bin/pycharm.svg
Exec=sh /data/opt/pycharm/bin/pycharm.sh
```

注意:向eclipse本身点击就能够打开，最后一行应该这样写
`Exec=/data/opt/eclipse`
保存，执行一下命令
`chmod a+x pycharm.desktop`

这时你会发现图标发生变化，然后把图标复制到桌面就能够使用。

**特别注意:**复制上面代码的时候注意每一行后面都不能有空格,很多同学制作失败就是因为copy上空格了,不过此时这个快捷方式还不能使用,你会发现图标并没有发生变化,双击也会出错.