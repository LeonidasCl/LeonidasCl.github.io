---
layout: post
title:  "git多人协作：一个too simple的入门教程"
date:   2016-08-03 20:02:57
categories: Git
comments: true
---

Github的一大主要优势就在于强大的多人协作，使用git而不进行多人协作，那和SVN有什么区别，相当于没用。

github用户通过fork他人的项目，建立自己的分支来参与开发。fork之后你会得到与被fork仓库一份一模一样的代码，各分支也会建立对应的关系。

在github上面浏览，找到想要fork的仓库后，点击进入仓库的页面，点开右边绿色的Clone or Download，得到仓库的地址：

![图片](http://obdvl7z18.bkt.clouddn.com/gh-pages/img/20160804/p1.jpg)

图中的网址`https://github.com/LeonidasCl/vita.git`就是这个仓库的地址。

选好想克隆的仓库后，找一个好地方，打开git bash，开始克隆，克隆操作会在这个地方建一个文件夹，那个文件夹就是你克隆下拉的仓库目录

USERNAME是该仓库原作者的用户名，REPONAME就是仓库名了

`$git clone https://github.com/USERNAME/REPONAME.git`

正常的克隆画风应该是这样：

	$git clone https://github.com/USERNAME/XXXX.git
	Cloning into `Spoon-Knife`...
	remote: Counting objects: 10, done.
	remote: Compressing objects: 100% (8/8), done.
	remove: Total 10 (delta 1), reused 10 (delta 1)
	Unpacking objects: 100% (10/10), done.

这样你就有了一个克隆出来的本地仓库，git bash来cd进去可以看见能够识别了，和你自己建的仓库完全一样，只不过关联的是别人的远程库。

有的人并不怎么想搞个大新闻，他们只是想克隆下来学习一个，提升自己的姿势水平。但是克隆完之后，如果作者又更新代码，他们的本地仓库就看不见更改，你叫他们重新克隆，他们又不高兴，怎么办？

克隆完之后啊，还要做一个配置，让自己的仓库与原仓库保持同步，这样原作者更新代码的时候我们不用重新克隆也可以更新，这叫做闷声大发财！怎么办呢？

这时候要把远程的upstream加进来。USERNAME是原作者的名字，upstream这个东西，技术好的人可能已经在Nginx里接触过了，这是废话，因为Nginx的upstream和git里的并没有什么关系。洋文好的人就知道，这个词意思是“上游”，在git里就是最新的提交了。upstream也可以理解为原项目的地址，我们获取原项目代码的更新就需要这个地址。

`git remote add upstream https://github.com/USERNAME/xxxxxx.git`

操作完之后查看所有分支，列出所有分支看一下，应该就能见到两个upstream分支：

	$git remote -v
	origin    https://github.com/USERNAME/YOUR_REPO.git (fetch)
	origin    https://github.com/USERNAME/YOUR_REPO.git (push)
	upstream  https://github.com/ORIGINAL/ORIGINAL_REPO.git (fetch)
	upstream  https://github.com/ORIGINAL/ORIGINAL_REPO.git (push)

以后我们想和原仓库同步呢，就切到仓库目录下开git bash，用fetch命令把别人的master分支拿过来，然后合并到自己的master分支，代码的更新就完成了：

这个流程新老司机应该都很熟悉，才对头：

拿来：

`$git fetch upstream`

切回自己的主分支:

`$git checkout master`

合并拿来的分支到自己的主分支去：

`$git merge upstream/master`

搞这个合并的时候注意了，你在主分支上的更改就要失效了，如果不想丢失改动成果的话事先把本地master保存到别的分支去。

现在话题扯回来了，要是有的人不仅想要看代码，还想参与开发，或者是一个团队在进行开发，多个用户需要去开发一套代码，怎么办？

从前面我们知道每个用户都从仓库建立者那克隆后，就有了各自的本地仓库，克隆出来的本地仓库与远程仓库的所有分支都自动关联，比如远程有一个master一个develop，那克隆到本地也是有一个master和一个develop，并且本地与远程的所有同名分支自动关联，这个自动关联的作用我还不是很清楚，不过有一点可以明确的，就是push的时候，如果本地库与远程库关联了，仓库名一样，就可以省略命令的格式，正规格式是`$ git pull <远程主机名> <远程分支名>:<本地分支名>`，git push hostname就够了，不用后面<remote>：<local>。

如果你想加入一个项目的开发，首先得在github上fork，这个没有什么好说的。fork完之后你的github主页上就有该项目的分支了，然后克隆到本地。怎么克隆之前已经讲了。克隆下来之后就在自己的分支下开始开发。举个例子，以著名的glide为例：

先去找到项目的主页去fork：

![图片](http://obdvl7z18.bkt.clouddn.com/gh-pages/img/20160804/p11.jpg)

然后就能在自己的主页看到这个项目，这时候在本地克隆。注意克隆的时候要从自己的远程库克隆（克隆地址的用户名要是自己的），不然就克隆了原作者的仓库，只能看代码不能提交了。

![图片](http://obdvl7z18.bkt.clouddn.com/gh-pages/img/20160804/p2.jpg)

克隆好之后，做一些更改。用git status可以看到已经添加了一个txt文件：

![图片](http://obdvl7z18.bkt.clouddn.com/gh-pages/img/20160804/p3.jpg)

然后把目录下所有的文件加入，并提交到本地的master分支（就是当前分支）

![图片](http://obdvl7z18.bkt.clouddn.com/gh-pages/img/20160804/p4.jpg)

然后推送到远程分支，这里当然是推送到自己的远程分支。如果认为是推送到原作者分支就是协作的概念还没搞清楚。

![图片](http://obdvl7z18.bkt.clouddn.com/gh-pages/img/20160804/p5.jpg)

之前已经说过，本地与远程同名分支会自动关联，所以push的命令可以简化成上面的样子。

接下来就是关键步骤了。现在我们github上的远程库glide和原作者的glide已经不同了，假设我们想给glide贡献代码，把自己的更改合并到原作者的仓库去，就要在github上发起一个pull request，在自己的仓库页面点击New Pull Request后会跳转到原作者的仓库页面去发起一个pull request：

![图片](http://obdvl7z18.bkt.clouddn.com/gh-pages/img/20160804/p6.jpg)

![图片](http://obdvl7z18.bkt.clouddn.com/gh-pages/img/20160804/p7.jpg)

当然我也不可能去真的发一个pull request到glide去，不过假设我发了，那么原作者就能看到我的pull request，去处理是否接受请求，如果接受，那我就为这个开源项目成功贡献了代码。

这种方式安全性很好了，但是在小团队小项目的情况下，更改可能很频繁，项目负责人老去处理pull request。我就不愿意这么搞，最好是参与项目的用户都能push到我的仓库，这样省事。其实这种方式的协作反倒比较容易实现，又举个例子，为了让我直接push代码到原仓库，项目作者首先在仓库的settings选项中找到collaborators，搜索我的用户名，向我发出邀请：

![图片](http://obdvl7z18.bkt.clouddn.com/gh-pages/img/20160804/s0.jpg)

![图片](http://obdvl7z18.bkt.clouddn.com/gh-pages/img/20160804/s01.jpg)

这时，我的github页面右上角会有提示，点开能找到这个邀请，我接受邀请之后就有权限推送到原仓库

![图片](http://obdvl7z18.bkt.clouddn.com/gh-pages/img/20160804/s011.png)

于是可以从项目作者那克隆代码到本地（注意这次克隆地址直接是项目作者的地址）：

![图片](http://obdvl7z18.bkt.clouddn.com/gh-pages/img/20160804/s1.jpg)

这次克隆，我的本地仓库关联的远程仓库直接就是原作者的远程仓库，我进行修改后，也能直接`git push origin`推送到原仓库去。

需要注意的是，这个例子为了方便都是在master分支上操作，实际的工作流不可能是这样的，一般来说远程库至少都会分主分支（master）和开发分支（develop），克隆下来的本地库也具备这两个分支，我们都是在本地develop分支上开发，push到远程的develop分支，再由项目负责人适时把远程develop合并到远程的master，最终发布的代码当然还是远程的master。