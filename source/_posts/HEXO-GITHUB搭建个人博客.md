---
title: HEXO+GITHUB搭建个人博客
date: 2018-07-10 21:46:51
tags: hexo
---

## 前言
之前就有意向想自己搭建一个个人博客，但因为总是觉得自己的知识还不够深厚，还不足以输出。但近来发现自己的生活过得相当堕落，心里的内疚感终于驱使我有重新燃起动力来完成搭建个人博客这一任务。原本想买一个云服务器来建站，无奈囊中羞涩，暂时还不忍付这笔额外的支出（如果以此来推测程序员的工资低那就成功被我误导了，我是个例，不具备代表性）。而hexo+github正好完美解决我的困扰，在搭建过程中遇到不少问题，都是在网上寻找答案，但也发现网上答案良莠不齐，有些已经比较老旧了，所以在这里就自己的经历为这一博客添砖加瓦，也期望能帮助一部分如我一样还在茫茫探索的同学。本文适用于Ubuntu系统。
<!--more-->

## 1. 安装HEXO

hexo是一款轻巧，强大的博客框架，它支持使用markdown语法来撰写博文，同时也能自由的选择各类漂亮的主题。hexo的安装也非常简单，但是在此之前你需要安装好以下两个以后也会经常用到的东西：

* Node.js

* Git

如果你的电脑已经安装好它们了，那么恭喜你，你直接可以安装hexo了，如果没有安装则按照以下步骤进行：

### 安装GIT 

`$ sudo apt-get install git-core`

### 安装node.js

如果按照hexo官网上的步骤直接安装，node.js的版本会比较旧，在后期安装hexo时就会报错。
更新Ubuntu

```
sudo apt-get update
sudo apt-get install -y python-software-properties software-properties-common
sudo add-apt-repository ppa:chris-lea/node.js
sudo apt-get update
```

安装node.js

```
sudo apt-get install nodejs
sudo apt install nodejs-legacy
sudo apt install npm
```

更新npm的包镜像源，将其换成淘宝源，这样方便快速下载

```
sudo npm config set registry https://registry.npm.taobao.org
sudo npm config list
```

全局安装n管理器（用于管理nodejs的版本）

```
 sudo npm install n -g
```

安装最新的nodejs（stable版本）

```
sudo n stable
# 查看node版本
sudo node -v
```
至此，我们终于可以安装hexo了

```
sudo npm install -g hexo-cli
```

## 2. hexo的本地运行

hexo安装好以后，运行以下命令来生成一个hexo的初始化本地文件夹，它也是hexo的工作区（`<folder>`是你自己命名的文件名）：

```
$ hexo init <folder>
$ cd <folder>
$ npm install
```
初始化完成以后，文件目录树是这样子的：
```
.
├── _config.yml
├── package.json
├── scaffolds
├── source
|   ├── _drafts
|   └── _posts
└── themes
```
其中`_config.yml`是配置文件，大部分设定都在这里完成，`package.json`是应用的一些数据，markdown语法的渲染也在这里，`themes`存放主题文件，`source`博客的主要内容就存放在这里。

初始化完成以后可以本地试运行以下：

```
$ hexo g    #生成静态网页
$ hexo s    #运行本地服务器
```
输入网址`http://localhost:4000/`就可以看到你的博客长什么样子了。

## 3. 更换主题

hexo的默认主题是landscape，但是大部分人认为其比较锋利，不太美观，所以换主题也是大部分人都会做的事。hexo有很多博客主题可以选用，具体可以参考一下[这篇文章](https://www.jianshu.com/p/bcdbe7347c8d)。我选用的是[yilia](https://github.com/litten/hexo-theme-yilia),那就以yilia为例来说说更换主题的步骤：
首先，进入hexo的本地文件夹（我的文件夹名为`blog`），下载主题

```
$ git clone https://github.com/litten/hexo-theme-yilia.git themes/yilia
```

修改hexo根目录下的 _config.yml ： theme: yilia

更新主题则用

```
$ cd themes/yilia
$ git pull
```
具体的主题配置可以参考[yilia](https://github.com/litten/hexo-theme-yilia)上的说明。修改完成以后可以再本地试运行一下，看一看效果。
```
$ hexo clean
$ hexo g
$ hexo s
```

## 4. 新建文章

首先Ctrl+C停止当前的本地服务，然后
```
hexo n  "我的第一篇文章"
```
这样就会在博客目录下source\_posts中生成相应的`我的第一篇文章.md文件`( 例如 /home/xx/blog/source_posts/我的第一篇文章.md ),然后可以通过支持markdown语法的编辑器来对文章进行编写，我用的是vscode+markdownlint插件。编写完成以后可以按照前述的步骤进行本地预览。

## 5.关联github

这篇文章有非常详细的介绍：[Ubuntu上用Hexo搭建博客托管到github](https://blog.csdn.net/lyb3b3b/article/details/78706077),这里我就不再多说。

## 6. 部署博客

前面的步骤完成以后，终于，到了见证成果的时候。首先，我们需要修改配置文件，即`blog/_config.yml`:

```
deploy:
type: git
repo: <repository url>
branch: [branch]
message: [message]
```
参数 | 描述
:-|:-:
repo | 库（Repository）地址
branch | 分支名
message | 自定义提交信息

例如：我的配置为

```
deploy:
  type: git
  repo: https://github.com/nanyan375/nanyan375.github.io.git
  branch: master
```
还需要一条命令
```
npm install hexo-deployer-git --save
```
用于使用deploy命令时调用git push，将更新信息推送至github.com。最后，终于可以推送了。
```
$ hexo clean
$ hexo g
$ hexo deploy
```

## 7. 参考文章与链接

* [Ubuntu16.04安装最新版nodejs](https://blog.csdn.net/u014361775/article/details/78865582)

* [Documentation of HEXO](https://hexo.io/docs/#What-is-Hexo)
