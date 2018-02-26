---
title: AKJ_UAV_APP调试问题及解决办法
date: 2017-09-26 09:17:00
tags:
- android
categories: 工作
---
  <font size="5" color="lightseagreen"> 在调试ARDRone parrot2.0飞机APP过程中遇到的一些问题及解决办法。
 </font>
<!--more-->

## [](#问题一、如何导入Vitamio库 "问题一、如何导入Vitamio库")问题一、如何导入Vitamio库

[解决办法： Vitamio 的导入与简单使用](http://blog.csdn.net/rd_w_csdn/article/details/53466372)
[Android Studio导入Vitamio多媒体开发框架](http://www.cnblogs.com/happyhacking/p/5365192.html)
但是按照上面方法还是没有导入成功,只差这一步：
在工程settings.gradle 添加`include‘：vitamio’`
如图：
![](http://ow7va355d.bkt.clouddn.com/Screenshot%20from%202018-01-20%2011-10-42.png)

## [](#问题二、关闭MIUI优化后，在调试APP时，会在手机端报问题，但手机变得非常卡顿。 "问题二、关闭MIUI优化后，在调试APP时，会在手机端报问题，但手机变得非常卡顿。")问题二、关闭MIUI优化后，在调试APP时，会在手机端报问题，但手机变得非常卡顿。

## [](#问题三：在APP中使用Vitamio视频播放插件时，在android5-0的手机使用没有问题，但是在小米5-android7-0的手机中出现以下问题 "问题三：在APP中使用Vitamio视频播放插件时，在android5.0的手机使用没有问题，但是在小米5,android7.0的手机中出现以下问题")问题三：在APP中使用Vitamio视频播放插件时，在android5.0的手机使用没有问题，但是在小米5,android7.0的手机中出现以下问题
<figure class="highlight java"><table><tr><td class="gutter"><pre><div class="line">1</div></pre></td><td class="code"><pre><div class="line">java.lang.UnsatisfiedLinkError: No implementation found <span class="keyword">for</span> <span class="keyword">void</span> io.vov.vitamio.MediaPlayer.native_init() (tried Java_io_vov_vitamio_MediaPlayer_native_1init</div></pre></td></tr></table></figure>

[解决办法：](https://github.com/yixia/VitamioBundle/issues/385)使用[Vitamio5.2](https://www.vitamio.org/Download/)

在使用5.2版本后出现以下问题：
<figure class="highlight java"><table><tr><td class="gutter"><pre><div class="line">1</div></pre></td><td class="code"><pre><div class="line">java.lang.UnsatisfiedLinkError: Expecting an absolute path of the library: libstlport_shared.so</div></pre></td></tr></table></figure>

解决办法还在上面的链接。
<figure class="highlight java"><table><tr><td class="gutter"><pre><div class="line">1</div><div class="line">2</div><div class="line">3</div><div class="line">4</div><div class="line">5</div><div class="line">6</div><div class="line">7</div><div class="line">8</div><div class="line">9</div><div class="line">10</div><div class="line">11</div><div class="line">12</div><div class="line">13</div><div class="line">14</div><div class="line">15</div><div class="line">16</div><div class="line">17</div><div class="line">18</div><div class="line">19</div><div class="line">20</div><div class="line">21</div><div class="line">22</div><div class="line">23</div><div class="line">24</div><div class="line">25</div><div class="line">26</div><div class="line">27</div><div class="line">28</div><div class="line">29</div><div class="line">30</div><div class="line">31</div><div class="line">32</div><div class="line">33</div><div class="line">34</div><div class="line">35</div><div class="line">36</div></pre></td><td class="code"><pre><div class="line">hi all, i use vitamio version <span class="number">5.2</span>.3 and get the same problem and i have tried all the above solutions and nothing works.</div><div class="line">then i <span class="keyword">try</span> to change the code bit on MediaPlayer.java</div><div class="line"></div><div class="line"><span class="keyword">if</span>(LIB_ROOT==<span class="keyword">null</span>)&#123;</div><div class="line">    	    	System.loadLibrary(<span class="string">"stlport_shared"</span>);</div><div class="line">    	    	System.loadLibrary(<span class="string">"vplayer"</span>);	</div><div class="line">    	    	loadFFmpeg_native(<span class="string">"libffmpeg.so"</span>);</div><div class="line">    	    	loadVVO_native(<span class="string">"libvvo.9.so"</span>);</div><div class="line">    	    	loadVVO_native(<span class="string">"libvvo.9.so"</span>);</div><div class="line">    	    	loadVAO_native(<span class="string">"libvao.0.so"</span>);</div><div class="line">    	    &#125;<span class="keyword">else</span>&#123;</div><div class="line">    	    	System.load(LIB_ROOT+ <span class="string">"libstlport_shared.so"</span>);</div><div class="line">    	    	System.load(LIB_ROOT+ <span class="string">"libvplayer.so"</span>);</div><div class="line">    	    	loadFFmpeg_native(LIB_ROOT+<span class="string">"libffmpeg.so"</span>);</div><div class="line">    	    	loadVVO_native(LIB_ROOT+ <span class="string">"libvvo.9.so"</span>);</div><div class="line">    	    	loadVAO_native( LIB_ROOT+ <span class="string">"libvao.0.so"</span>);      </div><div class="line">    	    &#125;</div><div class="line">and replace it with</div><div class="line"></div><div class="line"><span class="keyword">if</span>(LIB_ROOT==<span class="keyword">null</span>)&#123;</div><div class="line">    	    	System.loadLibrary(<span class="string">"stlport_shared"</span>);</div><div class="line">    	    	System.loadLibrary(<span class="string">"vplayer"</span>);	</div><div class="line">    	    	loadFFmpeg_native(<span class="string">"libffmpeg.so"</span>);</div><div class="line">    	    	loadVVO_native(<span class="string">"libvvo.9.so"</span>);</div><div class="line">    	    	loadVVO_native(<span class="string">"libvvo.9.so"</span>);</div><div class="line">    	    	loadVAO_native(<span class="string">"libvao.0.so"</span>);</div><div class="line">    	    &#125;<span class="keyword">else</span>&#123;</div><div class="line">                  System.loadLibrary(<span class="string">"stlport_shared"</span>);</div><div class="line">                  System.loadLibrary(<span class="string">"vplayer"</span>);</div><div class="line">                  loadFFmpeg_native(<span class="string">"libffmpeg.so"</span>);</div><div class="line">                  loadVVO_native(<span class="string">"libvvo.9.so"</span>);</div><div class="line">                  loadVVO_native(<span class="string">"libvvo.9.so"</span>);</div><div class="line">                  loadVAO_native(<span class="string">"libvao.0.so"</span>);</div><div class="line">    	    &#125;</div><div class="line">after that I <span class="keyword">try</span> in emulator android <span class="number">7.1</span>.1 and succeed</div><div class="line">I hope <span class="keyword">this</span> way also solve your problem :)</div></pre></td></tr></table></figure>