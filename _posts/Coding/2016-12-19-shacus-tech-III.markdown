---
layout: post
title:  "docker-拥抱docker hub"
date:   2016-12-19 17:56:29
categories: Coding
comments: true
---


追影技术·第二章：云时代的小型APP（三）
-------------------------------------

### docker-拥抱docker hub




　　DockerHub是一个类似GitHub的网站，在这个网站上有全球用户发布的各种容器供我们下载，加入dockerhub能让我们更快地进行容器知识的学习，全球开源贡献者的工作对我们的帮助是很大的。当然在变得足够强大之后我们也要成为一个开源社区的贡献者，去回报。

　　首先当然是要去dockerhub注册一个用户了，打开https://docs.docker.com去注册一个，填写完基本信息后我们的邮箱会收到验证邮件。点开验证邮件里的链接，就可以去登录了。

登录之后我们先创建一个仓库：


![图片1](http://obdvl7z18.bkt.clouddn.com/image/20161219/00.png)


![图片2](http://obdvl7z18.bkt.clouddn.com/image/20161219/01.png)

　　需要填写仓库名、简要和详细的描述信息，仓库的权限（公有或私有）等信息，填好这些基本的信息后点create就建立了一个docker仓库。这样通过浏览器创建的仓库默认都是公有的。



　　现在，我们再把之前本地创建的新镜像push到dockerhub上去，先用images命令看看要推的是哪个：

![图片2](http://obdvl7z18.bkt.clouddn.com/image/20161219/02.png)

　　显然，第一个镜像是之前创建的，也就是等会要推上去的，可以看到它的镜像ID是e5867970ccc4。我们来给它做个标记，用tag命令：


`$docker tag e586 leonidascl/docker-whale:latest`

　　ID并不一定要打完整，只要能够在列表里区分，docker就能找到。给镜像设置tag之后，我们就可以根据tag来操作这个镜像。



　　接着我们用docker login命令在本地登录一下dockerhub。输入命令后会提示输入用户名和密码。注意，这个登录命令要输入的是用户名而不是邮箱。这个步骤和在本地配置github账号也很相似的。

成功之后我们就可以用docker push命令去推送到远程仓库：


![图片2](http://obdvl7z18.bkt.clouddn.com/image/20161219/03.png)



　　往dockerhub进行push的过程非常慢，一看就知道是因为防火长城的存在。在这么慢的速度下进行实际开发是不可能的，好在国内已经有很多服务商在提供docker服务，我们可以用这些服务商提供的服务来快速构建持续集成的系统。