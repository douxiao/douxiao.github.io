---
title: python环境配置
date: 2018-12-26 09:00:21
tags:
- env
- python
categories: 工作
---
  

　　<font size="5" color="lightseagreen"> 这篇博客是总结Python环境配置问题，各种坑因为系统安装了好几个Python环境，系统默认的Python环境变量只有一个，在这里如果需要多个版本的Python,自己使用的虚拟环境，virtualenv。</font>
<!--more-->

**Linux如何执行命令**

Linux的终端输入任何一条命令，Shell都会去到几个和环境变量（$PATH）相关的文件中寻找对应的路径下是否有可以执行的文件。这些可执行的文件里放着的是我们输入命令和如何执行这些命令的说明。只有找到了这个文件shell才知道如何执行命令，这个过程可以简化理解为：
<figure class="highlight"><table><tr><td class="gutter"><pre><div class="line">1</div></pre></td><td class="code"><pre><div class="line">输入命令 -&gt; shell 读入命令 -&gt; 查找文件 -&gt; 定位文件中的环境变量 -&gt; 浏览环境变量对应的路径下的文件 -&gt; 在文件里寻找命令和执行方法 -&gt; 找到了，按照要求执行 -&gt; 找不到，输出找不到</div></pre></td></tr></table></figure>

**shell常用的和环境变量相关的命令**
<figure class="highlight python"><table><tr><td class="gutter"><pre><div class="line">1</div><div class="line">2</div><div class="line">3</div><div class="line">4</div></pre></td><td class="code"><pre><div class="line">$ env和$printenv用于展示环境变量配置文件的内容</div><div class="line">$ set展示所有的变量</div><div class="line">$ export将变量导出到接下来的程序环境中</div><div class="line">$ echo 输出如果跟着变量名，会把变量的值输出</div></pre></td></tr></table></figure>

**相关环境变量配置文件**
涉及到的文件

<figure class="highlight python"><table><tr><td class="gutter"><pre><div class="line">1</div><div class="line">2</div><div class="line">3</div></pre></td><td class="code"><pre><div class="line">/etc/profile</div><div class="line">/etc/bashrc</div><div class="line">/etc/zshrc</div></pre></td></tr></table></figure>

如果是etc目录下则是所有的用户共享的配置文件

<figure class="highlight python"><table><tr><td class="gutter"><pre><div class="line">1</div><div class="line">2</div><div class="line">3</div><div class="line">4</div></pre></td><td class="code"><pre><div class="line">~/.profile</div><div class="line">~/.bash_profile</div><div class="line">~/.bashrc</div><div class="line">~/.zshrc</div></pre></td></tr></table></figure>

**环境变量示例**

`export PATH=&quot;/usr/bin:/usr/local/bin&quot;`

export表示将变量导出到接下来的程序环境中。冒号是链接符号，这里的程序会顺序的逐个去找对应的执行文件。bin文件夹下一般存放的是可执行的二进制文件，当输入命令的时候，程序会逐个文件去找，如果找到了就会执行，找不到，就会反馈找不到。