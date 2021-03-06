---
layout: post
title:  "Android自定义View的使用"
date:   2016-06-26 00:56:37
categories: Android
comments: true
---

安卓开发中，进行界面编程时之前都是用已有的view来画界面，因为安卓提供的view已基本足够满足开发需求。但今天出现了这样的情况：产品和设计部门的界面设计图和各种资源要在一周后才能完成，在这之前开发没法写界面，要是所有开发人员等一周再开发，对于本来就很紧迫的开发周期无疑是雪上加霜。

这个时候才想到了先实现几个自定义view，其中包含一些常用的布局，到开发时可以复用，以提高开发的速度。

实现自定义view步骤可以比较简明地分为以下环节：

首先，实现一个自定义的view类，从布局管理器继承一个合适的，再自己加上需要的功能，我就加上了一个滑动动画。

![图片](http://obdvl7z18.bkt.clouddn.com/img/20160626p1.jpg)

对于这个view的一些属性值，在res的value里建一个XML来声明，以便在其它代码中调用：

![图片](http://obdvl7z18.bkt.clouddn.com/img/20160626p2.jpg)

想要使用这个view，当然是把它添加到某个activity或fragment的布局文件中，就像添加一个原有的view一样，添加的时候给这些属性赋上默认值，若代码中未设置，就会使用这些默认值了：

![图片](http://obdvl7z18.bkt.clouddn.com/img/20160626p3.jpg)

在该view的代码中这样获取上图fragment中的数值：

![图片](http://obdvl7z18.bkt.clouddn.com/img/20160626p4.jpg)

Getxxxxx的第二个参数是解决xml中没有搜索到对应值的情况的，那时第二个参数就会被赋给这个属性。

至此，自定义view就添加完成。Excited！

复用的时候，只要在复用的XML里添加这个view的项，再写明需要的值就可以了，如果按下图这种写法就不能复用，一开始没有理解透彻，觉得用typearray转一圈取数值很麻烦，所以写了这样的代码：

![图片](http://obdvl7z18.bkt.clouddn.com/img/20160626p5.jpg)

这样会使自定义的view类失去可重用性。所以写view是时候一定要多留心，规范的写法远比简单的写法要好。