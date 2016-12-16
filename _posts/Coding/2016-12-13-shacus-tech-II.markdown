---
layout: post
title:  "docker-先做一个微小的尝试"
date:   2016-12-15 19:38:46
categories: Coding
comments: true
---


追影技术·第二章：云时代的小型APP（二）
-------------------------------------

### docker-先做一个微小的尝试



　　首先我们尝试在一台用作测试的Linux机子上安装docker。这台机子是最基础的配置（1核CPU，1G内存），此前机子上已经安装了WDCP管理系统并运行LNMP框架，资源使用率如下：

![图片1](http://obdvl7z18.bkt.clouddn.com/img/20161215/01.png)



Docker目前仅支持64位系统，Linux内核要在3.10以上，先检查是否满足条件：

`$ uname -r`
`3.10.0-229.el7.x86_64`

然后我们给系统升级一下，确保最新状态:

`$ yum update`

再添加docker仓库的地址：

`$ tee /etc/yum.repos.d/docker.repo <<-'EOF'`

在这个文件中输入：

`[dockerrepo]`
`name=Docker Repository`
`baseurl=https://mirrors.tuna.tsinghua.edu.cn/docker/yum/repo/centos7`
`enabled=1`
`gpgcheck=1`
`gpgkey=https://mirrors.tuna.tsinghua.edu.cn/docker/yum/gpg`

然后保存退出！Excited！这样就可以开始安装docker了：

`$ yum install docker-engine`

现在可以在服务器上启动docker服务：

`$ systemctl enable docker.service`

成功之后，就可以启动docker了：

`$ systemctl start docker`



通过WDCP管理后台的进程列表我们可以看到docker服务端已经启动了：

![图片2](http://obdvl7z18.bkt.clouddn.com/img/20161215/02.png)

　　启动的这个docker叫docker deamon，是一个服务端，还没有运行任何容器，docker deamon是用来管理这台母机上运行的所有容器的（可以回想一下守护进程的概念）。

再给docker服务加个开机自动启动，方便一些：

`$ systemctl enable docker`



​	启动好了我们就来运行一下，首先当然是要运行具有重大意义的hello-world镜像。当然这时候本地并没有这个镜像，docker会去下载来运行。

![图片3](http://obdvl7z18.bkt.clouddn.com/img/20161215/03.png)

​	解释一下这条命令 docker run --rm hello-world,docker当然就是告诉系统要使用docker服务了，docker run命令用于运行或创建并运行一个镜像，--rm参数是clean up，会在容器运行结束后清除文件数据——一个容器里仅会运行一个程序，当那个程序结束时容器也会被终止，rm参数就是在容器运行结束后把它产生的数据都清理掉，我们临时运行一个容器的时候可以加这个选项。我们运行这个命令后，docker首先会去找叫hello-world的镜像，发现没有，就到docker-hub上去下了一个来，然后把这个镜像装到一个容器里面运行（前面提到的镜像就是容器打包的单位），程序运行完退出的时候对应容器也会停止。




​	用 docker ps可以查看当前运行的容器，加上a参数查看所有运行记录（包括已经结束的容器），如果我们在docker run时加上-rm参数则容器结束后用ps命令查不到记录，因为该容器结束后数据都被清空了。如下图，只能看到不带rm参数时运行的hello-world记录：

![图片4](http://obdvl7z18.bkt.clouddn.com/img/20161215/04.png)

​	

​	接下来我们从docker hub上下载一个镜像来玩玩，docker hub有点像github，正如docker命令有点像git命令一样。Dockerhub上有海量的各种镜像供用户下载，我们也可以贡献自己的镜像上去，这是在dockerhub上搜到的whalesay镜像：

![图片5](http://obdvl7z18.bkt.clouddn.com/img/20161215/05.png)



我们把这个镜像下载下来运行：

`$ docker run docker/whalesay cowsay boo`



命令后面的cowsay boo是给whalesay程序的命令，运行一下就知道是什么了：

![图片6](http://obdvl7z18.bkt.clouddn.com/img/20161215/06.png)



接下来我们用docker images可以看到本地的所有镜像里已经有了刚才下的这个：

![图片7](http://obdvl7z18.bkt.clouddn.com/img/20161215/07.png)

我们接下来可以对这个镜像做微小的修改，自己写一个dockerfile用来创建自定义镜像。



先建一个目录，我们会把构建镜像所需的所有文件都包含进来

`$ mkdir mydockerbuild`

进入目录创建dockerfile

`$ cd mydockerbuild`
`$ touch Dockerfile`

在这个文件里加入这三行：

`FROM docker/whalesay:latest`
`RUN apt-get -y update && apt-get install -y fortunes`
`CMD /usr/games/fortune -a | cowsay`

​	FROM语句说明这个镜像基于原来的whalesay创建，RUN语句是说明这个镜像需要安装的软件，第三行加入的一条CMD语句让容器启动时运行该命令。这里提一下，CMD语句在Dockerfile中只能有一条，如果多了就只运行最后一条。



然后开始build，看到命令后面的.是不是觉得和git命令又更相似了：
`$ docker build -t docker-whale .`
从即时输出中我们一目了然的看到正在执行哪一步构建：

![图片8](http://obdvl7z18.bkt.clouddn.com/img/20161215/08.png)



　　成功结束后一个叫docker-whale的镜像就会被build出来，用docker images命令就可以看到：

![图片9](http://obdvl7z18.bkt.clouddn.com/img/20161215/09.png)



接下来运行一个，这样我们就完成了第一个自己构建的镜像了！excited！

![图片10](http://obdvl7z18.bkt.clouddn.com/img/20161215/10.png)