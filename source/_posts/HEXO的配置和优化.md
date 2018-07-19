---
title: HEXO的配置和优化
date: 2018-07-19 10:15:02
tags:
---
#### 1. 前言

之前已经用github+hexo搭建了一个博客，并选用yilia主题来优化界面，其实基本的功能已经具备了，如果你只是纯粹的想把它当做是印象笔记一样来使用，那当然也是可以的。不过我相信，大部分同学折腾博客的初衷就是展示自己，结交好友。如果从这个角度来说，那么评论功能是一定要实现的。今天就着这些内容来和大家探讨一下。
<!--more-->

#### 2. 不同终端的同步

如果只能有一台终端可以部署博文，那就太不git了。既然选用了github，那实现不同终端之间的同步自然不会是难事。主要思路是在你自己博客的仓库新建一个分支，并将之设为默认分支，然后将hexo的源码文件上传到这一分支，当登录不同终端时就可以`clone`博客的源码到本地环境了。以下就是以我的博客为例来说明具体操作步骤。
- 先在远程仓库新建一个branch，比如hexo
![PMBmQI.png](https://s1.ax1x.com/2018/07/14/PMBmQI.png)
- 将hexo的分支设为默认
[![PMBMef.md.png](https://s1.ax1x.com/2018/07/14/PMBMef.md.png)](https://imgchr.com/i/PMBMef)
- 将我们的源文件上传到hexo分支
```
$ git init
$ git add -A
$ git commit -m "blog源文件"
$ git branch hexo
$ git remote add origin git@github.com:yourname/yourname.github.io.git
$ git push origin hexo -f
```
>注意这里有个巨大的坑！！！如果你用的是第三方的主题theme，是使用git clone下来的话，要把主题文件夹下面把.git文件夹删除掉，不然主题无法push到远程仓库，导致你发布的博客是一片空白

#### 3. gitment实现评论功能

关于评论功能有很多产品可供选择，大部分hexo主题也都为常见的几种产品预留了配置内容，我用的主题yilia在配置文件里支持的包括多说，网易云跟帖，畅言，disqus，gitment。其中，多说已经关闭，畅言需要网站备注，网易云跟帖我没有使用过，不过继多说关闭之后国内评论的相关组件也都是风雨飘摇。我选用的是gitment，它的作者说明如下：

>Gitment 是作者实现的一款基于 GitHub Issues 的评论系统。支持在前端直接引入，不需要任何后端代码。可以在页面进行登录、查看、评论、点赞等操作，同时有完整的 Markdown / GFM 和代码高亮支持。尤为适合各种基于 GitHub Pages 的静态博客或项目页面。

从说明来看完美契合我们的需要，它需要以下步骤来完成：

##### 一. 注册 OAuth Application

[点击此处](https://github.com/settings/applications/new)来注册一个新的 OAuth Application。注册时要填的内容以我的博客为例：
![PMaxzt.jpg](https://s1.ax1x.com/2018/07/14/PMaxzt.jpg)
（上图为我已经注册完毕后的场景，所以它的按钮是update application）
注册完毕后，一定要记住Client ID和Client Secret.

##### 二. 安装gitment

在你博客的根目录下安装gitment:
```
$ npm install --save gitment
```

##### 三. 配置gitment

gitment安装完毕以后，接下来就是在主题的_config.yml完成以下配置：
```
gitment_owner: false      #你的 GitHub ID
gitment_repo: ''          #存储评论的 repo
gitment_oauth:
  client_id: ''           #client ID
  client_secret: ''       #client secret
```
其中的client_id，client_secret就是上文注册OAuth Application产生的。

##### 五. 初始化评论

页面发布后，你需要访问页面并使用你的 GitHub 账号登录（请确保你的账号是第二步所填 repo 的 owner），点击初始化按钮。之后其他用户即可在该页面发表评论。
如果在此时出现如下图所示的错误：
[![PM0Jr6.md.png](https://s1.ax1x.com/2018/07/14/PM0Jr6.md.png)](https://imgchr.com/i/PM0Jr6)
则多半是Authorization callback URL填写错误。

#### 4. 不蒜子实现站内统计

[不蒜子](http://busuanzi.ibruce.info/)与百度统计谷歌分析等有区别：“不蒜子”可直接将访问次数显示在网页上（也可不显示）；对于已经上线一段时间的网站，“不蒜子”允许初始化首次数据,而且不蒜子的使用方式也极其简单，正如它的官网所说，极简风格。只需要简单地插入两段代码即可完成所有的设置。对于我的博客来说，yilia主题似乎没有考虑到站内统计的要求（确实，现在好像大家都不愿向外显示过多的数据，特别是在数据不好看的时候)。所以我的单篇文章的访问量的统计我不知道应该放在哪里，而且我对前端的东西也是不太了解，不能自己定制格式，所以目前我的博客只设置了总的站点访问统计。步骤很简单，只要将以下代码
```
<script async src="//dn-lbstatics.qbox.me/busuanzi/2.3/busuanzi.pure.mini.js">
</script>
```
放置在`themes/你的主题/layout/_partial/footer.ejs`添加上述脚本即可，当然你也可以添加到 header 中。
然后将需要显示的标签
```
<span id="busuanzi_container_site_uv">
  本站访客数<span id="busuanzi_value_site_uv"></span>人次
</span>
```
也放在`themes/你的主题/layout/_partial/footer.ejs`中即可。

#### 5. 参考链接
1. [Gitment：使用 GitHub Issues 搭建评论系统](https://imsun.net/posts/gitment-introduction/)

2. [不蒜子](http://ibruce.info/2015/04/04/busuanzi/)

3. [搭建hexo博客并简单的实现多终端同步](https://righere.github.io/2016/10/10/install-hexo/)
