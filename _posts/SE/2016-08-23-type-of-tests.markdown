---
layout: post
title:  "Android测试【一】Android开发中测试的种类"
date:   2016-08-24 00:41:50
categories: SE
comments: true
---


安卓开发中单元测试是很重要的，AS项目目录中就有专门的测试路径，搞好测试的好处不需要多提，每个熟悉软件工程的人都知道。单元测试主要针对最小可重用的代码单元进行——通常是某个方法，或某个UI组件。道理大家都懂，可是做起来就不是大家都乐意了。在安卓下进行测试可能有点麻烦，测试代码量也不少，但这是必须的技能之一。

在安卓开发中，单元测试大概可分为两类：

### 1.本地测试

这种测试是在本地的JVM虚拟机环境下运行的，不需要安卓的环境，也就不访问安卓的各种类（但可以模拟这些类）。

### 2.UI测试

这个叫Instrumented tests，可以在这样的测试中访问安卓类，对UI进行操作，比如模拟用户点击、输入等。



下面先来说一下本地测试。



因为不需要运行app，通过本地测试来检验代码可以节省我们大量时间。通常用Junit+Mockito搭建本地测试框架，Mockito是用来模拟各种相关类的测试工具，用它来模拟某些与要测试代码有关联类，可以让我们只关注要测试的那部分代码而不需要真的创建关联类的对象。



首先随便建个新项目，找到我们app的build.gradle模块，在dependences节点下添加这两个依赖库：

```
dependencies {
    // JUnit 4 测试框架
    testCompile 'junit:junit:4.12'
    // Mockito 测试框架
    testCompile 'org.mockito:mockito-core:1.10.19'
}
```

重新构建一次项目后我们就可以开始写测试类了。值得一提的是，之前写过一篇Juint3测试框架的，现在我们用Junit4，这个测试框架更新之后比原来高到不知道哪里去了，既不需要继承TestCase类也不需要一个一个地写测试方法，用Junit4，我们大可一边谈笑风生一边写测试类。



在Android视图下我们来切换到单元测试，打开BuildVariants设置页，将测试的类型换成UnitTests。现在AndroidStudio不会自动识别单元测试类了，所以我们得在测试目录下创建测试类。

![图片](http://obdvl7z18.bkt.clouddn.com/img/201608242/01.jpg)

![图片](http://obdvl7z18.bkt.clouddn.com/img/201608242/02.jpg)

可以看见test目录下已经有一个测试类了。方法前加了@Test符号的就是测试方法，可以看下图，这个方法什么事都没干，只是查了个2+2是不是等于4，不过也算是个例子了。右键点击test目录或该类都可以运行测试。



如果我们想要测试的类与安卓类有关联，那就得用Mockito模拟了，现在我们就来测试一下SecondActivity里的一个Adapter。首先找到这个adapter类右键-goto-tests去创建测试类：

![图片](http://obdvl7z18.bkt.clouddn.com/img/201608242/03.jpg)

会弹出测试类创建的选项窗口，这时候当然要选Junit4，其它的都可以默认：

![图片](http://obdvl7z18.bkt.clouddn.com/img/201608242/04.jpg)

```java
@RunWith(MockitoJUnitRunner.class)
public class ListviewAdapterTest {

    //设成常量好判断
    private static final String FAKE_STR = "TITLE";

    @Mock//有这个关键字就是模拟的类
    SecondActivity secondActivity;//模拟的类
    Context mockContext;//也是模拟的

    @Test
    public void testAdapter(){
        //把数据初始化一下，
        List<Listitem> lst=new ArrayList<>();
        Listitem listitem=new Listitem();
        listitem.setTitle(FAKE_STR);
        lst.add(listitem);

        //开始测试，ListviewAdapter是真正要测试的类，所以肯定得创建实例的
        SecondActivity.ListviewAdapter adapter = secondActivity.new ListviewAdapter(mockContext,lst);

        //检验结果
        assertThat(adapter.getItem(0).getTitle(),is(FAKE_STR));
    }

}
```

写完这段代码，右键test下的绿色目录java，选择run all tests with coverage，这可以让我们看到测试代码的覆盖率，这时候测试就会提示是否添加覆盖率数据的窗口，选中间add：

![图片](http://obdvl7z18.bkt.clouddn.com/img/201608242/05.jpg)

我们还可以把测试的详细数据也显示出来：

![图片](http://obdvl7z18.bkt.clouddn.com/img/201608242/06.jpg)

![图片](http://obdvl7z18.bkt.clouddn.com/img/201608242/07.jpg)

可以看到上上图用时总计0.854s，上图为单个方法的用时，0.012s。

而coverage窗口可以看到我们测试代码的覆盖率：

![图片](http://obdvl7z18.bkt.clouddn.com/img/201608242/08.jpg)

这样就完成了单元测试。

需要注意的是，我们模拟那些类，并不是因为不能创建它们的对象，而是因为我们要测试的不是它们——在团队开发时你只需要给自己负责完成的类写单元测试，集成测试则由专人负责。模拟出来的类，单元测试时是不管其正确性的，而new出来的则不同，会真的创建一下试试看。下图，在测试类里加一行错代码，被秒发现了：

![图片](http://obdvl7z18.bkt.clouddn.com/img/201608242/09.jpg)

到现在相信我们已经了解单元测试的基本用法了。