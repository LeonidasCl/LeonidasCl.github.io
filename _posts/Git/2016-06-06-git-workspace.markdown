---
layout: post
title:  "Git工作区、暂存区、仓库目录的概念"
date:   2016-06-06 20:50:11
categories: Git
comments: true
---

工作区，可以理解为一个工作目录，比如.git文件夹所在的这个文件夹就是一个工作区

![图片](http://obdvl7z18.bkt.clouddn.com/img/20160606p1.jpg)

通常我们在这个工作区里直接进行代码的编辑，文件的修改。

.git文件夹是在git init命令后创建的，是git版本库。暂存区、当前分支都在这个版本库里。

对于工作目录中新建的文件，执行git status时会提示该文件untracked

![图片](http://obdvl7z18.bkt.clouddn.com/img/20160606p2.jpg)

对已有的文件作出更改，执行git status时会提示

![图片](http://obdvl7z18.bkt.clouddn.com/img/20160606p3.jpg)

这两种情况都要先git add，再git commit提交更改，即先将更改保存到暂存区，再将暂存区的更改提交。

暂存区是一个保存有下次将要提交的文件列表的文件，每当我们执行git commit命令，暂存区的所有更改就会被提交到仓库目录，文件就会得到更新。我们在对代码进行修改后，先要用git add命令将这个修改加入暂存区，再执行git commit将修改提交，才算完成到git仓库目录的存储。

需要注意的是，git的版本控制是以保存文件修改的方式而不是保存文件副本的方式，也就是说如果我们对文件A进行第一次修改，然后对其执行git add，再对A进行第二次修改，然后执行git commit，这时候真正被提交到git仓库目录的只有第一次修改——因为git add只添加了第一次修改的暂存。

对于某个特定文件而言，截至上一次取出，如果未修改，就是已提交(committed)状态，如果修改过，未加入暂存区，就是已修改(modified)状态，已修改并放入暂存区，就是已暂存(staged)状态。git中任何文件，都会处于这三种状态之一。（untracked是这个文件根本没加入版本库）

仓库目录就是保存当前分支数据的地方了，clone的时候就是复制这个目录的文件。本地commit的时候也是将更改提交到这个目录。

工作区、暂存区、仓库目录的关系可以用这张图说明

![图片](http://obdvl7z18.bkt.clouddn.com/img/20160606p4.jpg)

这样git仓库目录中的文件一共可以有三种状态：已修改、已暂存、已提交。

*对修改的管理

`$ git checkout -- filename` 可以撤销工作区的更改，如果执行这个命令时该文件已暂存，就回滚到上一次add的状态，如果未暂存，就回滚到上一次commit的状态

`$ git reset HEAD filename` 可以撤销暂存区的更改，执行这个命令后工作区的更改仍存在，用git status仍能看到在工作区有何改动，并重新add