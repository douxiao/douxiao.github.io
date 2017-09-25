---
title: 搭建Hexo+github博客(二)
date: 2017-09-23 22:31:56
tags: 
- hexo
- github
categories: 博客
comments: true
---
<font size=5 color=darkturquoise> 如何在不同电脑上同时写hexo博客？</font>
<!--more-->

<font size=5 >前言</font>
 
 
 　　Hexo通过将Markdown文件编译成html文件，然后将html文件直接部署到网站上，所以被称作静态博客，因为直接就是访问的最终的html文件，不会有PHP那样的中间处理，所以对浏览器来讲会比较快。但是这样在不同的电脑上写博客的时候，从github上pull下来，只能够得到一些静态页面，而且没有源md文件,所以在每次提交博客部署的时候，就会把原来的博客清空。
  　　为了完美解决这个问题，可以**利用github的不同分支来分别保存网站静态文件与hexo源码（md原始文件及主题等）**，实现在不同电脑上都可以自由写博客，这样不好之处就是别人能得到你的博客源文件。
　　
　　**   注：假设你有工作PC和个人PC，你原有的博客搭建在个人PC上，以下首先介绍个人PC上的操作步骤。**

# <font size=5 >个人PC</font>


## <font size=4 color=darkturquoise> 在github上新建远程仓库</font>

　　将原来的page项目删除，新建一个和原来名字一样的空项目。不用初始README.md，此时只有一个空的master分支。
　　**本地的目录不要动**，到时候需要用你原来博客的配置和文章替换该项目中的文件夹。你可以把你本地的博客目录复制一份作为备份，以免误操作将原来的内容进行改动。
## <font size=4 color=darkturquoise> 本地初始化一个Hexo项目
</font>　

　　新建一个空目录，比如我的文件名为DxBlog,作为你的博客目录。进入该目录，执行以下命令:
```
hexo init
npm install
npm install hexo-deployer-git --save
```
　　然后用自己原来博客里的文件替换掉这里的<font color=red>source\ ,  scaffolds\ , themes\ , _config.yml </font> 替换成自己原来博客里的。注意，**这里需要把themes/yelee中的.git/目录删除**，因为在安装博客主题的时候是直接git下来的，会有原始的git信息，如果不删除的话，推送到hexo分支后yelee目录为空，自己的电脑是linux系统./git目录是看不到的　需要用命令行：`sudo rm -rf .git/`删除。**还有如果之前更改过node_modules文件中的东西同样需要更改，比如球形标签云。**
## <font size=4 color=darkturquoise> 将整个目录推送到master
</font>

　　要推送到master分支，首先要将该目录初始化为本地Git仓库：
  ```
git init
//把博客目录下所有文件推送到master分支
git remote add origin git@github.com:douxiao/douxiao.github.io.git
git add .　//添加当前工作目录文件到index
git commit -m "some comments" //生成一个commit
git push origin master     //推送服务器
```
## <font size=4 color=darkturquoise> 在github上新建一个分支
</font>

　　新建一个分支hexo(名字可以自定义)，这时候hexo分支和master分支的内容一样，都是hexo的源文件。**并把hexo设为默认分支，这一步需要在你的库设置里完成**，这样的话在另外一台机器上克隆下来就直接进入hexo分支，并且以后所有操作都是在hexo分支下完成。
　　为什么需要这个额外的分支呢？
　　因为`hexo d`只把静态网页文件部署到master分支上，所以你换了另外一台电脑，就无法pull下来继续写博客了。有了hexo分支的话，就可以把hexo分支中的源文件(配置文件、主题样式等)pull下来，再hexo g的话就可以生成一模一样的静态文件了。
## <font size=4 color=darkturquoise> 部署博客</font>

　`hexo g -d`
　　博客已经成功部署到master分支，这时候到github查看两个分支的内容，hexo分支里是源文件，master里是静态文件。
　　**注意：网站根目录下的_config.yml配置文件中branch一定要填master，否则hexo d就会部署到hexo分支下**
## <font size=4 color=darkturquoise> 关联到远程hexo分支</font>  
　　在本地新建一个hexo分支并与远程hexo分支关联：
  ```
git checkout -b hexo
git pull origin hexo
```
　　另外别忘了，如果有修改的话，要推送到hexo分支上去：
```
git add .
git commit -m  ""
git push origin hexo

```
# <font size=5 >工作PC</font>

　　个人PC上的工作已经完成了，下面讲一下如果你换到了另外一台电脑上，应该如何操作。
　　将博客项目克隆下来,<font color=red> 这里特别注意以后的操作需要在这个克隆下来的文件下操作，因为里面有git信息，在之后的git push才能正确完成，自己在这里踩坑了。</font>
```
git clone git@github.com:douxiao/douxiao.github.io.git
```
　　克隆下来的仓库和你在个人PC中的目录是一模一样的，所以可以在这基础上继续写博客了。但是由于.gitignore文件中过滤了**node_modules**\，所以克隆下来的目录里没有node_modules\，这是hexo所需要的组件，所以要在该目录中重新安装hexo，但不需要hexo init。
```
npm install -g hexo-cli
npm install
npm install hexo-deployer-git --save
```
　　**注意：这里一般还需要安装一些一些必须的插件，就是说可靠像之前在个人pc上安装的那些插件一样。**

　　新建一篇文章测试
```
hexo new "work PC test"
```
　　推送到hexo分支
```
git init 
git add .
git commit -m "add work PC test"
git push origin hexo
```
　　部署到master分支
```
hexo g -d
```
# <font size=5 >日常操作</font>

　　如果上面的过程都操作无误的话，你就可以在任何能联网的电脑上写博客啦。一般写博客的流程是下面这样。
**写博客前**
　　不管你本地的仓库是否是最新的，都先pull一下，以防万一：

```
git pull origin hexo
```
　　**在这里自己遇到的问题是**：每次在git pull之前，需要执行命令 `git stash`
执行完git pull后需要执行　｀git stash pop｀，[Git:代码冲突常见解决方法](http://blog.csdn.net/iefreer/article/details/7679631)，[git stash用法](http://www.cppblog.com/deercoder/archive/2011/11/13/160007.aspx)
　　把最新的pull下来，再开始撰写新的博客。
　　写博客
```
hexo new "title"
```
　　然后打开source/_posts/title.md，撰写博文。
　　写完博客
　　先推送到hexo分支上：
```
git add .
git commit -m "add article xxx"
git push origin hexo
```
　　**在这里自己遇到的问题是**：Git-Permission denied (publickey)解决办法因电脑而异。自行谷歌。

　　最后部署到master分支上：
```
hexo g -d
```

# <font size=5 >常见问题解决</font>


1. [配置hexo 为什么运行到 hexo server 这步就没用了？](https://www.zhihu.com/question/28847824)
2. [Hexo—正确添加RSS订阅](http://hanhailong.com/2015/10/08/Hexo%E2%80%94%E6%AD%A3%E7%A1%AE%E6%B7%BB%E5%8A%A0RSS%E8%AE%A2%E9%98%85/)
3. [如何在不同电脑上同时写hexo博客？](http://chown-jane-y.coding.me/2017/03/15/%E5%A6%82%E4%BD%95%E5%9C%A8%E4%B8%8D%E5%90%8C%E7%94%B5%E8%84%91%E4%B8%8A%E5%90%8C%E6%97%B6%E5%86%99hexo%E5%8D%9A%E5%AE%A2%EF%BC%9F/)
4. [删除本地SSH并新建一个ssh](http://blog.csdn.net/yemoweiliang/article/details/53215979)
5. [在github上添加SSH key的步骤：](http://www.cnblogs.com/ayseeing/p/3572582.html)
6. [Git-Permission denied (publickey)
](https://stackoverflow.com/questions/2643502/git-permission-denied-publickey)






























