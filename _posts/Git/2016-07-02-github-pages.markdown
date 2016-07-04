---
layout: post
title:  "资瓷一个github pages，把博客迁移过来"
date:   2016-07-02 01:03:22
categories: Git
comments: true
---

由于新浪云和独立域名纷纷涨价，我已经决定了，把博客迁移到github pages上去，本来还想给域名apply for record继续用，但新浪云说，你这个apply for record啊就要交什么什么材料，而且中央已经研究决定了，备案是不容易的。政策的决定权是很重要的，所以我的博客就到了github。迁移到github上也没有什么别的，主要就做三件事：

##第一，建立了本地和远程的gh-pages仓库

Github上已经说的很清楚了，首先要建一个仓库，名字就叫`username.github.io`，username是自己的用户名，这个仓库就是存放网页静态文件的地方。创建完成后再把这个仓库克隆到本地，以便在本地编辑后推送到远程库。熟悉git的应该很快就完成这两步了。建好仓库后，在根目录写一个index.html，推上去，访问`username.github.io`就可以看到这个页面已经被解析了。当然，我们这样子是不行的，还是too simple，需要用Jekyll来强化一下这个静态网站的功能。

##第二，安装jekyll框架

什么是Jekyll呢，简单说就是一个站点的generator，就是解析静态文件并能够生成站点的工具。网上已经有很多Jekyll站点的模板了，为了专注内容，直接下一个模板来改改就可以作自己的博客了，多么简单，但是搞出来的这个东西呀，excited！

##第三，也就是最重要的，没有预览的一律不准发上网

在本地写博客push到远程仓库就完成了个人博客的更新，虽然可能会有几分钟延迟，但也是极好的了。不过为了减少不必要的推送次数，还是要在本地先预览自己的修改，预览好后在push啊，这是坠吼的！要想在本地预览，就得先搭个本地的Jekyll环境。

首先去ruby官网去装个ruby，现在用的windows系统，所以我就下了个ruby installer来安装。Ruby安装好后再安装Bundler，这是用来管理ruby的gem的工具，所以我们可以用bundler来安装和运行Jekyll。安装bundler，命令行下启动ruby后运行

`gem install bundler`

安装可能会慢的一逼，装好以后到博客的代码仓库根目录下去建一个文件叫gemfile，文件内容是这两行，确定一下有关配置，有了它就能安装Jekyll了：

`source 'https://rubygems.org'`

`gem 'github-pages', group: :jekyll_plugins`

完成后运行`bundle install`去安装一个，装完本地环境就搭建完成了。

因为我已经下载了一个Jekyll网站的模板，所以这时候切换到我的博客根目录，用`bundle exec jekyll serve`命令启动Jekyll服务器，再访问`localhost:4000`就可以看到网站了。

每次更改后都要运行一次，_site文件夹中的文件就是最近运行时生成的站点文件。