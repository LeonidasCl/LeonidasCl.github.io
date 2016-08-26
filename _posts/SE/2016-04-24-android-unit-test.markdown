---
layout: post
title:  "分享几种Android单元测试的姿势"
date:   2016-04-24 13:23:10
categories: SE
comments: true
---

首先要明白什么是单元，我的理解是“单元”在这里可以指软件的模块、类、甚至单个方法——联想软件重用的定义，重用粒度可以小至代码级别（直接复制粘贴），可大至完整的软件模块，类比到单元测试的定义,“单元”就是我们想要测试的代码、类、模块了。不过常用的单元测试应该是对单个方法的测试。

建立一个Android项目后可以看到这两个文件路径：

![图片](http://obdvl7z18.bkt.clouddn.com/img/20160424p1.jpg)

其中AndroidTest中是需要Android设备才能运行的单元测试的目录，test中是可以直接运行的单元测试代码目录。

具体要使用哪一个，用户可以自行切换：

![图片](http://obdvl7z18.bkt.clouddn.com/img/20160424p2.jpg)

根据我们使用Junit版本的不同有两种测试的方法，这里借用stackoverflow上的一个图：

![图片](http://obdvl7z18.bkt.clouddn.com/img/20160424p3.jpg)

当我们使用JUnit3时需要继承TestCase并在需要测试的方法名前加test前缀（否则不能被识别为测试代码），而使用JUnit4时只需要用@Test的notation即可。需要注意的是，如果既继承了TestCase又使用@Test是会报错的。

测试一个类的步骤可以分为实例化、测试方法、检验结果。

在Junit4中@Before标注的方法会在所有测试方法运行前都要运行，可以用来初始化要测试的对象。对应的，在Junit3中通过重载setUp方法来实现统一的初始化。对应的，@After是所有测试方法运行完之后会运行的方法。

对想要测试的类右键点击类名，goto->test，会弹出一个对话框，可以选择要测试的方法和使用的Junit版本。生成的一个使用Junit4的测试类如下：

![图片](http://obdvl7z18.bkt.clouddn.com/img/20160424p4.jpg)

右键点击类名运行即可测试

![图片](http://obdvl7z18.bkt.clouddn.com/img/20160424p5.jpg)

测试结果，绿色表示符合预期

![图片](http://obdvl7z18.bkt.clouddn.com/img/20160424p6.jpg)

用robotium框架测试安卓应用：
-------------------------------

1.安装robotium

![图片](http://obdvl7z18.bkt.clouddn.com/img/20160424p7.jpg)

2.重启AS后，打开要测试的项目，并在Tools 菜单中找到 Robotium Recorder，弹出的选择窗口中选择当前的app（也可以点击select apk选择已经build完成的apk文件，注意app必须是已签名的）

![图片](http://obdvl7z18.bkt.clouddn.com/img/20160424p8.jpg)

3.点击下一步后，出现的窗口中点击settings进行设置

![图片](http://obdvl7z18.bkt.clouddn.com/img/20160424p9.jpg)

`Use sleeps`选项建议网络交互较多的应用勾选，这样可以让rpbotium以相同的速度录制。

`Keep app data`就是app数据再启动新的录制时是否保存。 

`Identify class over string`建议勾选，勾选后将使用String而不是id来识别类。

`Click and drag coordinates`拖拽和点击屏幕坐标是否录制。

设置完成后点击`new robotium test`

![图片](http://obdvl7z18.bkt.clouddn.com/img/20160424p10.jpg)

录制完成后点stop，再点save，输入名字即可保存新的录制。

重新sync project，会发现原项目的兄弟目录下多了一个包，里面包含的就是刚才测试生成的代码，想要进行测试再运行一遍这个包就可以了。

用Espresso框架测试安卓应用：
-----------------------------

1.在应用的build.gradle里加入espresso依赖库：

`
	dependencies {
		...
		androidTestCompile 'com.android.support.test.espresso:espresso-core:2.2.1'
		'com.android.support.test:runner:0.5'
	}
`

在build.gradle的defaultConfig中添加：

`testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"`

2.执行sync project，不报错的话espresso就添加完成了。

如果报错，可以参照官网给的样例配置文件：

Build.gradle中

![图片](http://obdvl7z18.bkt.clouddn.com/img/20160424p11.jpg)

![图片](http://obdvl7z18.bkt.clouddn.com/img/20160424p12.jpg)

Build.gradle的defaultConfig中

![图片](http://obdvl7z18.bkt.clouddn.com/img/20160424p13.jpg)

![图片](http://obdvl7z18.bkt.clouddn.com/img/20160424p14.jpg)

回到前文的类中写一些测试代码，用来测试登录功能是否正常：

![图片](http://obdvl7z18.bkt.clouddn.com/img/20160424p15.jpg)

显然，onView方法是获取UI控件，获取控件后调用perform方法对控件进行操作，或是调用check进行检查，写完后就可以测试这些控件是否按预想状况运行了。

关于espresso的各个测试方法，官网有一张图详细说明：
图片太大了，所以只放个链接，在[**这里**][link1]

[link1]: https://google.github.io/android-testing-support-library/docs/espresso/cheatsheet/index.html