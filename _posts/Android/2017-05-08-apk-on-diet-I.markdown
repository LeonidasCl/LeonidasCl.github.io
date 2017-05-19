---
layout: post
title:  "安卓APK瘦身（一）APK文件的结构说明"
date:   2017-05-08 13:35:26
categories: Coding
comments: true
---

发布一个体量精简的APP对于用户转化是至关重要的，很多APP并没有什么功能，安装包却要十几兆，用户根本懒得下载。对安装包进行瘦身有多种手段，效果也非常显著，下面我们就来实践一次对apk的瘦身。本章节涉及到的安装包例子是我们的“追影”APP。

在进行瘦身之前我们需要了解一个APK安装包会包含哪些内容。展示确定答案之前来做一个简单的推测：代码肯定是有的，美术资源也一定包含，此外还有各种第三方库和.so库等等依赖文件。接下来我们就打开apk文件来查看其结构。其实APK文件就是一个zip压缩包，我们可以用AS直接打开它：



![图片1](http://obdvl7z18.bkt.clouddn.com/img/20170508/01.jpg)

上图可以看到构成apk的各部分文件及它们的绝对大小、相对大小（百分比），至于整个包的大小，RawFileSize表示apk在磁盘上的大小，DownloadSize则是从谷歌应用市场上下载的大小。其中classes文件就是编译好的代码，其中都是java源码生成的字节码文件，但是注意这些字节码JVM是没法执行的，因为DVM和JVM的虚拟机标准并不兼容，所以并不能用反编class文件的工具来反编这些dex文件。

![图片2](http://obdvl7z18.bkt.clouddn.com/img/20170508/02.jpg)



首先来看.dex文件，如下图，这个apk有两个.dex文件，说明采用了multi-dex技术。为什么要这么做呢，那是因为我们的app方法总数已经超过了64k的dex文件方法数上限，报UNEXPECTED TOP-LEVEL EXCEPTION了，我就在app的build.gradle中做了以下配置，以开启multi-dex，这样做之后apk中就有两个.dex（classes和classes2），他们都被添加到dexpathlist中以保证能够被虚拟机加载，当然multi-dex技术的使用里面有很多的坑，在这里不展开了。

![图片3](http://obdvl7z18.bkt.clouddn.com/img/20170508/03.jpg)

![图片4](http://obdvl7z18.bkt.clouddn.com/img/20170508/04.jpg)



接下来是res，看下图，res是美术资源和xml布局文件，可以看到在这个APP里占空间明显过大，尤其是drawable文件中的图片需要精简大小。

![图片5](http://obdvl7z18.bkt.clouddn.com/img/20170508/05.jpg)



下图，lib是.so本地库，里面都是封装好的本地方法。当然这些本地方法是不能在AS查看了，都是第三方开发好的功能，需要一些相关工具来解析才能查看。

![图片6](http://obdvl7z18.bkt.clouddn.com/img/20170508/06.jpg)



resources.arsc是以二进制的形式储存资源信息的文件，相当于是资源的一个索引，AS能查看这个文件里保存的资源信息，包括资源ID、类型、位置等。

![图片7](http://obdvl7z18.bkt.clouddn.com/img/20170508/07.jpg)



下图是assets文件，包含部分美术资源和配置文件。与resources.arsc文件不同的是assets文件中的资源没有资源ID，打包时这assets资源直接放进apk，不做额外的处理。

![图片8](http://obdvl7z18.bkt.clouddn.com/img/20170508/08.jpg)



这里的AndroidMainfest文件则是我们整个app所有的AndroidMainfest合并而成的；

![图片9](http://obdvl7z18.bkt.clouddn.com/img/20170508/09.jpg)



META-INF中包含的是签名文件。

![图片10](http://obdvl7z18.bkt.clouddn.com/img/20170508/10.jpg)



从图中就可以知道主要是.dex、lib、res文件在占用空间，精简apk的大小肯定要从这些文件做起。