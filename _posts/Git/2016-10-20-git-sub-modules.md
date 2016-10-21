---
layout: post
title:  "git开发中子模块相关操作"
date:   2016-10-20 20:10:06
categories: Git
comments: true
---

​	在git多人协作时遇到这样一个问题：由于服务端（我们是tornado）的一些限制，导致前后端代码都要放到服务端代码仓库里去（网页等静态文件必须放到tornado的static目录里去，否则url映射会出错报404），前后端开发不得不同时去维护一个代码仓库，十分混乱，估计调试的时候会错漏百出——前端更改代码后，push之前不得不从远程合并与自己无关的服务端代码，反之服务端也一样。这样子搞是不行的。

​	为了解决这个问题，需要在服务端的仓库中添加一个子模块来保存前端代码。前端人员维护前端代码，看起来和维护独立的代码仓库没有区别，后端人员只要定期去拉取子模块的更新，代码就可以跑了。子模块这个机制有点类似于依赖管理工具，但功能弱一些，不过实现前后端人员并行在父模块和子模块互不干扰地开发当然是没有任何的问题，这一点是可以肯定的。

​	简单介绍之后就来实际动手添加子模块。现在假设服务端的仓库叫做gitlearn1016，里面已经有服务端代码了，而客户端仓库叫做XXXXXXXXchat，也已经有客户端代码了，我现在就进到服务端仓库目录去，添加一个子模块：

`git submodule add 仓库地址`

如果想自定义路径（比如static），直接加在仓库地址后面就可以了，就和克隆时自定义路径的方法一样

![图片](http://obdvl7z18.bkt.clouddn.com/gh-pages/img/20161020/01.jpg)

从bash的输出就可以知道，添加子模块的一个工作就是把子模块克隆到父模块下，当然不仅仅是克隆，submodule add还一并把子模块信息写进了父模块仓库目录的.gitmodules文件里，打开这个文件我们就能看到所有子模块的信息：

![图片](http://obdvl7z18.bkt.clouddn.com/gh-pages/img/20161020/03.jpg)

这个时候看status，可以看到添加了两个文件，分别就是刚刚说的.gitmodules和子模块的仓库目录文件夹xxxxxxxxchat

![图片](http://obdvl7z18.bkt.clouddn.com/gh-pages/img/20161020/02.jpg)

然后我们把添加好子模块的服务端代码commit一下：

![图片](http://obdvl7z18.bkt.clouddn.com/gh-pages/img/20161020/04.jpg)

注意这里的mode，子模块提交的模式是160000，这种模式与我们通常提交的模式并不一样，在这个模式下git不会跟踪其中的内容，只会记住其发生了更改。

怎么能够验证一个呢？我们把父模块和子模块的代码都做微小的改动，然后在父模块执行

`git add .`

易知，这条命令会非常实实在在的把所有更改加入暂存，我们这时候看status

![图片](http://obdvl7z18.bkt.clouddn.com/gh-pages/img/20161020/06.jpg)

在status的输出中，我们又非常实实在在的看到确实只有父模块的更改被加入暂存了，子模块的更改并没有被跟踪，所以我们在父模块push的时候，子模块是不会被一并更新的。这就满足了父模块的开发人员不必管理子模块代码的需求了。如下图，父模块push代码后，子模块的更新仍未跟踪：

![图片](http://obdvl7z18.bkt.clouddn.com/gh-pages/img/20161020/08.jpg)

父模块开发人员是不用麻烦了，子模块开发人员呢，更不用麻烦。此时子模块开发人员只要自顾自的add、commit、push，将子模块代码更新就是了。我这个例子是子模块也在本地改的，所以就进到子模块目录去更新，实际中肯定是在另一台机子上，不过操作和这里一点区别都没有，子模块维护人员可以像不存在父模块时一样更新子模块代码。

![图片](http://obdvl7z18.bkt.clouddn.com/gh-pages/img/20161020/09.jpg)

弄清楚了父模块子模块的开发人员是如何提交的，就可以模拟一下他们是如何更新的，现在我把远程仓库的子模块代码改了——模拟子模块开发人员在另一台机子上对子模块进行更新。来到父模块的目录，更新这个子模块：

`git submodule update --remote shacuswechat`

当remote后面什么都不加的时候会更新所有的子模块。这个更新是把子模块远程的commit都抓到本地来，在下图可以看到，更新完之后status，显示子模块有新的commits

![图片](http://obdvl7z18.bkt.clouddn.com/gh-pages/img/20161020/10.jpg)

我们可以用git diff非常实实在在的看到子模块有什么新的提交：

![图片](http://obdvl7z18.bkt.clouddn.com/gh-pages/img/20161020/11.jpg)

这个时候进到子模块去开bash，发现当前分支是一个commit id，当然这样不能push。造成这个的原因是我们在父模块直接更新子模块，相当于只fetch没有merge，子模块的master和当前这个head并没有任何的关系。这个head没有建立与历史分支的关系，就把它叫做游离的head。我们解决这个也简单，就是切回到master并把head合并过来就可以了。

![图片](http://obdvl7z18.bkt.clouddn.com/gh-pages/img/20161020/12.jpg)

![图片](http://obdvl7z18.bkt.clouddn.com/gh-pages/img/20161020/13.jpg)

这样先更新再进子模块的操作实际上是比较麻烦的，我们的一个简便手段就是，update的时候加上merge参数，自动就把子模块的更新合并。

![图片](http://obdvl7z18.bkt.clouddn.com/gh-pages/img/20161020/14.jpg)

这样更新完后，进子模块可以看见已经合并好了。

![图片](http://obdvl7z18.bkt.clouddn.com/gh-pages/img/20161020/15.jpg)