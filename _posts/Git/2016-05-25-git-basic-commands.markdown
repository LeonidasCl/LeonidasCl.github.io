---
layout: post
title:  "Git基本操作"
date:   2016-05-25 22:48:54
categories: Git
comments: true
---

下载Git:
[https://git-scm.com/][link]

安装完成后，桌面上添加gitBash快捷方式作为命令行工具

输入命令可显示版本号，说明安装好了

![图片](20160525p1.jpg)

创建新目录并将其移到本地git数据库

`$ mkdir JunitTest`

`$ cd JunitTest`

`$ git init`

然后把把代码复制进文件夹,用

`$ git status`

检查目录的状态，发现新的文件

![图片](20160525p2.jpg)

把所有文件加入索引

![图片](20160525p3.jpg)

再次检查发现已加入索引，提交

![图片](20160525p4.jpg)

修改后再用status可检查状态

用Diff命令可详细查看更改的地方

对二进制文件是不能查看的，只能知道是否更改

![图片](20160525p5.jpg)

查看提交历史

![图片](20160525p6.jpg)

退回上个版本
![图片](20160525p7.jpg)

HEAD表示当前版本HEAD^是上一个版本

`$ git reset --hard XXXXXX`可以回到某个commitID的版本

在github上建一个空的库

关联本地库与远程库并将本地文件push到远程

![图片](20160525p8.jpg)

认证时用户名密码就是github的用户名密码

于是Github上已经有了这个库

![图片](20160525p9.jpg)

本地作了提交后，还需要用

`$ git push origin master`

把本地master分支的修改同步到github



总结：


建立本地git仓库

`$ mkdir 仓库名`

`$ cd 仓库名`

`$ git init`

查看更改

`$ git status`

提交

`$ git commit -m “说明”`

变更版本

`$ git reset --hard XXX`

查看所有修改

`$ git diff`

查看某文件修改

`$ git diff 文件名`

关联本地与远程仓库

`$ git remote add origin git@github.com:xxxxx/xxxxx.git`

推送到远程仓库（第一次同步要在push后加-u参数）

`$ git push origin master`

从远程仓库克隆到本地（可以用于fork其它项目之后的克隆）

`$ git clone git@github.com:xxxxx/xxxxx.git`


[link]:https://git-scm.com/