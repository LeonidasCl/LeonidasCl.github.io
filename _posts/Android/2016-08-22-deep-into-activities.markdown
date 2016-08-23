---
layout: post
title:  "四大组件：深入理解Activity【一】"
date:   2016-08-22 00:22:17
categories: Android
comments: true
---


作为Android应用的四大组件之一，Activity又通常是我们最先接触到的组件，这个组件对新手可以说是坠重要的。熟练运用Activity要求熟悉其生命周期，各种不同的加载模式，灵活地相互传递数据等等。今天先讲Activity的生命周期。

下面这张图就是Activity的生命周期图

![图片](http://obdvl7z18.bkt.clouddn.com/img/20160822/01.jpg)

来解释一下这张图：图中一共有七种方法，就是这七种方法贯穿Activity的整个生命周期。注意图中方法间的提示文字，从这张图我们可以总结出：当Activity创建后执行onCreate()->onStart()->onResume()后进入运行状态；此时，如果用户启动其它的Activity，这个Activity的onPause()->onStop()就会被调用。接下来注意了（敲黑板），onStop这画出来三条线，左边表示用户启动了其它APP，为了节省内存系统把原APP进程杀掉了，这样再回到原Activity时当然要重新创建了；中间一条，洋文已经写得很明白了，两种情况会导致Activity进入onDestroy():Activity处于finishing状态或者被系统销毁。判断是不是处于finishing状态可以用Activity的isFinishing方法来获取，但有一点可以明确的，就是如果处于finishing状态一定是调用了finish()方法，或者是你按了手机上的back键（效果等同调用finish）；至于被系统销毁，通常是因为那个Activity属于某个在后台运行的程序，且长期未使用。

上面一段可能略微复杂，不过图上是很清楚—————一旦调用了onDestroy()，再切回原Activity来就只能重新onCreate了，这个时候保存的状态（滚动的进度、输入框内容什么的）就会丢失了。

初步了解Activity生命周期了是不是能脑补出多个Activity切换时的情况了？我想还是不行的，要彻底看懂上图还得引入任务和返回栈的概念。任务和返回栈其实指的是同一个东西，但叫“任务”的时候并不强调顺序的概念，叫“返回栈”的时候就有顺序了（栈顺序）————任务就是安卓系统中的一组Activity（可以属于同一APP，也可以属于不同APP），返回栈就是这组Activity按栈顺序排列起来（栈顶是当前显示的Activity）。

返回栈如何工作呢？举个例子，用户启动一个APP，首先启动了ActivityA，然后又从ActivityA中启动了ActivityB，这个时候ActivityA就会从running状态执行到onStop，ActivityB会从launched状态执行到running。这个时候栈的结构是栈底为A，栈顶为B，我们按手机上的back键，ActivityB会执行onDestroy()并被销毁（出栈），ActivityA会走onStop()右边的那条线onRestart()->onStart()->onResume()到running，栈里又只剩下ActivityA，就是这样的一个过程。