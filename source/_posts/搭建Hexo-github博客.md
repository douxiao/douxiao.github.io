---
title: 搭建Hexo+github博客(一)
date: 2017-09-23 19:58:24
tags: 
- hexo
- github
categories: 博客
---
　　<font size=5 color=lightseagreen> 　这篇文章是总结自己的博客搭建之路，包括：初始识Hexo、配置博客的环境、初始化博客、域名申请、绑定域名、博客主题的选定、优化配置博客、markdown编辑器的选定、工作和个人笔记本同时写博客，工作电脑是linux系统，个人电脑是windows系统，这篇文章主要是按照Linux系统的安装布置博客。可能会分几篇博文来写。</font>
<!--more-->

# <font size=5 >**一、初识Hexo**</font>
　　　[Hexo](https://hexo.io/zh-cn/docs/index.html)是一款基于Node.js ，快速、简洁且高效的博客框架。Hexo 使用 Markdown（或其他渲染引擎）解析文章，在几秒内，即可利用靓丽的主题生成静态网页。
# <font size=5 >**二、安装Git环境**</font>  
　　　配置git前，需要到[github官网](http://www.github.com)注册自己的账号,然新建一个username.github.io的仓库。
　　　[git安装教程](https://git-scm.com/book/zh/v1/%E8%B5%B7%E6%AD%A5-%E5%AE%89%E8%A3%85-Git)
　　　[git配置教程](https://git-scm.com/book/zh/v1/%E8%B5%B7%E6%AD%A5-%E5%88%9D%E6%AC%A1%E8%BF%90%E8%A1%8C-Git-%E5%89%8D%E7%9A%84%E9%85%8D%E7%BD%AE)
　　　[git生成ssh公钥](https://git-scm.com/book/zh/v1/%E6%9C%8D%E5%8A%A1%E5%99%A8%E4%B8%8A%E7%9A%84-Git-%E7%94%9F%E6%88%90-SSH-%E5%85%AC%E9%92%A5)
   ```
$ git config --global user.name "John Doe"
$ git config --global user.email johndoe@example.com
```

# <font size=5 >**三、安装Node.JS环境**</font>
  　　　1. [Node.j安装下载地址](https://nodejs.org/en/)
  　　　2. [在Ubuntu下安装Node.JS的不同方式](https://linux.cn/article-5766-1.html)
  　　　3. 我使用的是按照命令行的方式，（如果是windows环境，直接下载软件安装就可以了，不过安装后需要重启电脑才能够生效）代码如下
```
$ apt-get install nodejs
$ npm -v
   5.4.2
$ npm -v
v8.4.0
```
注意：用命令行安装的版本一般不是很高，需要进行升级，升级教程:[一行命令搞定node.js 版本升级](http://www.16boke.com/article/detail/26)
# <font size=5 >**四、[安装Hexo](https://hexo.io/zh-cn/docs/setup.html)**</font> 
```
$ npm install -g hexo-cli
```
　　安装 Hexo 完成后，请执行下列命令，Hexo 将会在指定文件夹中新建所需要的文件。
```
$ hexo init <folder>
$ cd <folder>
$ npm install
$ hexo new test
$ hexo g    #生成静态文件
$ hexo s    # 启动服务器。默认情况下，访问网址为： http://localhost:4000/。
```
　　现在来介绍常用的Hexo 命令

　　npm install hexo -g #安装Hexo
　　npm update hexo -g #升级 
　　hexo init #初始化博客
　
　　命令简写
　　hexo n "我的博客" == hexo new "我的博客" #新建文章
　　hexo g == hexo generate #生成
　　hexo s == hexo server #启动服务预览
　　hexo d == hexo deploy #部署

　　hexo server #Hexo会监视文件变动并自动更新，无须重启服务器
　　hexo server -s #静态模式
　　hexo server -p 5000 #更改端口
　　hexo server -i 192.168.1.1 #自定义 IP
　　hexo clean #清除缓存，若是网页正常情况下可以忽略这条命令

**部署推送网站**

　　下一步将我们的Hexo与GitHub关联起来，打开站点的配置文件_config.yml，翻到最后修改为：

　　deploy: 
　　type: git
　　repo: 这里填入你之前在GitHub上创建仓库的完整路径，记得加上 .git
　　branch: master参考如下：
　　保存站点配置文件。

　　其实就是给hexo d 这个命令做相应的配置，让hexo知道你要把blog部署在哪个位置，很显然，我们部署在我们GitHub的仓库里。最后安装Git部署插件，输入命令：
  
  ```
  $ npm install hexo-deployer-git --save
  $ hexo clean 
  $ hexo g 
  $ hexo d
  ```
　　访问网站：usrname.github.io
  
# <font size=5 >**五、[注册并绑定域名](http://www.jianshu.com/p/cea41e5c9b2a)**</font>
　　
 　　[域名访问博客](http://www.douxiao.org)

# <font size=5 >**六、更换主题及优化**</font>

　1.博客默认的是landscape主题。
　2.自己使用了一段时间next,next主题挺不错的自己也配置了很久，[Next主题配置教程官网](http://theme-next.iissnan.com/theme-settings.html)，[Next主题优化](https://juejin.im/post/58eb2fd2a0bb9f006928f8c7).
　3.自己使用的主题是Yelee主题。
  　　[Yelee主题使用说明](http://moxfive.coding.me/yelee/)
  　　[Yelee主题增加球形标签云](http://www.netcan666.com/2015/12/15/Hexo%E4%B8%AA%E6%80%A7%E5%8C%96%E7%90%83%E5%BD%A2%E6%A0%87%E7%AD%BE%E4%BA%91/#u6548_u679C_u56FE)
  　　[Yelee主题其他优化](http://ngudream.com/2017/01/24/n-hexo-blog/)
　　[Yelee主题字数统计](http://www.flametao.cn/2017/01/29/wordcount/)
　4.Yelee主题增加红心，鼠标点击小红心的设置
　　将 [love.js](http://7u2ss1.com1.z0.glb.clouddn.com/love.js) 文件下载添加到 hexo/themes/yelee/source/js 文件目录下。
　　找到 hexo/themes/yelee/layout/_partial/footer.ejs 文件， 在文件的最后， 添加以下代码：
  ```
  <!--页面点击小红心-->
<script type="text/javascript" src="/js/love.js"></script>

```
  　5.[增加来必力评论系统](http://jomsou.me/2017/07/16/livere/) 















































