---
layout: post
title:  "Android测试【二】Android的自动化UI测试"
date:   2016-08-25 21:13:20
categories: SE
comments: true
---

​	前一篇介绍了本地测试，那是一种轻便快捷，易于实现的测试方式。但很多时候我们是需要测试UI的，我们通过代码模拟UI操作来实现这个测试，Espresso框架在这样的测试中使用广泛，今天就来介绍一下用Espresso进行UI测试。

​	首先要给我们的项目添加Espresso依赖包，还是一样，打开app模块的build.gradle文件，在其中的dependences添加依赖，首先给dependencies节点添加依赖：

```
    //espresso测试框架
    androidTestCompile 'com.android.support.test.espresso:espresso-core:2.2.2'
    //JUnitRunner运行器，包含了JUnit4支持
    androidTestCompile 'com.android.support.test:runner:0.5'
```

​	然后给android节点的defaultConfig节点添加运行器的配置：

```
testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"
```

​	sync project，结果gradle报了一警告：

![图片](http://obdvl7z18.bkt.clouddn.com/img/20160825/01.jpg)

​	这什么意思呢？意思是我们的app运行版本和测试版本用的某个依赖库版本不一样，这样可能造成的后果就是我们测试的时候app的表现与实际运行的时候不一样，这样子搞是不行的！然而这个support-annotations库我并没有显示声明它，只好强制统一。运行版本的是24.0.0，测试版本的是23.1.1，我们按照新版本来统一，在dependencies节点里加上一句：

```
    androidTestCompile 'com.android.support:support-annotations:24.0.0'
```

​	这样再构建一次项目就没有报错了。

​	接下来简单介绍怎么使用Espresso，用之前别忘了在build variants窗口把测试类型换成Android Instrumention Tests。同样我们看到目录里已经有一个测试类，这个类比本地测试的样例还要简单，没有做任何操作。于是我们就来使用Espresso。

![图片](http://obdvl7z18.bkt.clouddn.com/img/20160825/02.jpg)

​	首先来看下我们要测试的app是什么样子的，我要测试下面几个页面：主界面依次点击顶部三个按钮，然后开新的Activity，检验其按钮上的文字，然后返回。

![图片](http://obdvl7z18.bkt.clouddn.com/img/20160825/03.jpeg)

​	创建一个测试类的方法就不提了，还是一样的选中类名右键--goto--tests，记得用junit4。创建好后我们来到MainActivityTest，写上测试代码：

（为了使效果明显，每个操作后都加了wait方法，实际测试中除了网络请求等耗时操作外不需要加）

```
@RunWith(AndroidJUnit4.class)
public class MainActivityTest {

    //测试规则
    @Rule
    public ActivityTestRule<MainActivity> activityRule = new ActivityTestRule<>(
            MainActivity.class);

    @Test
    public void testOfMain() throws InterruptedException {
        //点击顶部第一个按钮
        onView(withId(R.id.btn_a)).perform(click());
        wait(2000);
        //检查新界面是否出现了登录按钮
        onView(withId(R.id.tv_login)).check(matches(withText("登录")));
        wait(2000);
        //点击顶部第二个按钮
        onView(withId(R.id.btn_b)).perform(click());
        wait(2000);
        //查看中心的图片是否显示
        onView(withId(R.id.imgview_3)).check(matches(isDisplayed()));
        wait(2000);
        //点击顶部的第三个按钮，显示计算器界面
        onView(withId(R.id.btn_c)).perform(click());
        wait(2000);
        //滑动到底部的加号按钮并点一下
        onView(withId(R.id.scrollFooter)).perform(scrollTo(),click());
        wait(2000);
    }
}
```

​	同样右键该目录或改类运行测试，不过测试前先开模拟器或连调试真机，这时候就能看到界面在被代码控制，直到测试完成。到了开发后期，测试代码多了起来，我们就可以休息一会，但是为了缓解浓浓的睡意，最好泡一杯Espresso坐等——没错，Espresso是个意大利语音译单词，意思就是浓咖啡。

关于Espresso的更多资料，可以移步[**这里**][link1]去查看，我想它并没有被墙。

[link1]: https://google.github.io/android-testing-support-library/docs/index.html