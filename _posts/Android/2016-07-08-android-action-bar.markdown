---
layout: post
title:  "ActionBar的花式使用"
date:   2016-07-08 22:50:36
categories: Android
comments: true
---

动作条（ActionBar），就是平时显示在activity顶部的那个组件，通常放置一些该activity重要的活动。为了使APP更美观，功能更多样化，最好定义自己的动作条。

从Android3.0也就是API11开始，所有使用了默认主题的APP都会有一个默认的动作条，这个默认的动作条不在我们的VIEW体系中，不好控制，再加上默认动作条对于不同Android版本表现不一，所以最好用Toolbar代替动作条，这也是谷歌推荐的做法。

首先在AndroidMainfest文件中声明一个不包含动作条的主题，这样就可以使用我们自定义的动作条了：

    <application
        android:name="com.example.pc.test.APP"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:theme="@style/AppTheme.NoActionBar" >

其中`AppTheme.NoActionBar`是在style.xml中自己定义的主题，也是继承Android原有的一个没有动作条的主题：
    
style.xml

    <style name="AppTheme.NoActionBar">
        <item name="windowActionBar">false</item>
        <item name="windowNoTitle">true</item>
        <!--溢出菜单样式 -->
        <item name="actionOverflowMenuStyle">@style/OverflowMenuStyle</item>

    </style>

    看上面代码可知，我在这里还定义了新的溢出菜单样式，使得溢出菜单风格和APP整体统一。

    现在可以写toolbar的布局了，建一个XML文档把Toolbar写在里面，要一个标题居中的：

    toolbar_layout.xml

    <?xml version="1.0" encoding="utf-8"?>
    <android:android.support.v7.widget.Toolbar
                                       xmlns:app="http://schemas.android.com/apk/res-auto"
                                       xmlns:android="http://schemas.android.com/apk/res/android"
                                       android:id="@+id/m_toolbar"
                                       android:background="@color/gold"
                                       android:elevation="4dp"
                                       android:theme="@style/ThemeOverlay.AppCompat.ActionBar"
                                       app:popupTheme="@style/ThemeOverlay.AppCompat.Light"
                                       layout_height="?attr/actionBarSize"
                                       layout_width="match_parent">
    
                                        <TextView
                                        android:id="@+id/m_toolbar_title"
                                        style="@style/TextAppearance.AppCompat.Widget.ActionBar.Title"
                                        android:layout_width="wrap_content"
                                        android:layout_height="wrap_content"
                                        android:layout_gravity="center"/>
    
        </android:android.support.v7.widget.Toolbar >`

然后在Activity的布局里include这个Toolbar：

        <include
        layout="@layout/toolbar_layout"
        android:layout_width="match_parent"
        android:layout_height="?attr/actionBarSize"
        android:gravity="center"/>

接下来在Activity中使用这个动作条，要做这么几件事

首先获取到这个Toolbar并将其设置为这个activity的appbar：

       mtoolbar = (Toolbar) findViewById(R.id.m_toolbar);
        //将这个toolbar设置为actionBar
        setSupportActionBar(toolbar);

然后把原有的标题显示关掉。为了能让标题居中显示只好这么做了，自带的标题也是TextView，难以获取到，即使可以通过遍历的方式去获取，拿到之后也不能设置居中。appbar其实是一个ViewGroup，用代码获取layoutparams的话只获取到了ViewGroup的，设置不到居中，我暂时没发现什么办法，所以把原有标题显示关掉，自己写了TextView作为标题并居中。

       actbar=getSupportActionBar();
        if (actbar!=null)
        //关闭标题显示
        actbar.setDisplayShowTitleEnabled(false);`

然后建一个xml文档在里面写好溢出菜单：

    <?xml version="1.0" encoding="utf-8"?>
    <menu xmlns:android="http://schemas.android.com/apk/res/android"
          xmlns:app="http://schemas.android.com/apk/res-auto">
    
    
        <item
            android:id="@+id/action_favorite"
            android:icon="@drawable/login_password"
            android:title="@string/action_favorite"
            app:showAsAction="never"
            />
    
    
        <item android:id="@+id/action_settings"
              android:title="@string/action_settings"
              app:showAsAction="never"/>
    
    </menu>

在activity里记得重载渲染溢出菜单的方法：
  
    @Override//不要忘记渲染APP动作条的overflow菜单
    public boolean onCreateOptionsMenu(Menu menu) {
        getMenuInflater().inflate(R.menu.menu_appbar, menu);
        return true;
    }

    这样一来是完成了，还缺少的就是美化工作，QQ的顶部动作条就是和设备状态栏融合的，它们在不同设备上会渐变透明（4.4+）或半状态栏透明（5.0+），这叫沉浸式的状态栏，解决方案已经很成熟了，现在就把代码贴上来：

    activity的OnCreate中：（必须是继承AppCompatActivity的）,除了这段设置，activity的顶级布局中也要加上`android:fitsSystemWindows="true"`属性，以调整系统窗口布局适应自己的布局，保证自己的view（contentview）不被动作条遮挡住。

       Window window = getWindow();
        //4.4版本及以上
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.KITKAT) {
            window.addFlags(WindowManager.LayoutParams.FLAG_TRANSLUCENT_STATUS);//状态栏透明
            window.addFlags(WindowManager.LayoutParams.FLAG_TRANSLUCENT_NAVIGATION);//导航栏透明
            //创建状态栏的管理实例
            SystemBarTintManager tintManager = new SystemBarTintManager(this);
            //激活状态栏设置
            tintManager.setStatusBarTintEnabled(true);
            //设置状态栏颜色
            tintManager.setTintResource(R.color.gold);
            //激活导航栏设置
            tintManager.setNavigationBarTintEnabled(true);
            //设置导航栏颜色
            tintManager.setNavigationBarTintResource(R.color.gold);
        }
        //5.0版本及以上
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
            window.clearFlags(WindowManager.LayoutParams.FLAG_TRANSLUCENT_STATUS
                    | WindowManager.LayoutParams.FLAG_TRANSLUCENT_NAVIGATION);//状态栏透明
            window.getDecorView().setSystemUiVisibility(View.SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN
                    | View.SYSTEM_UI_FLAG_LAYOUT_HIDE_NAVIGATION
                    | View.SYSTEM_UI_FLAG_LAYOUT_STABLE);//导航栏透明
            window.addFlags(WindowManager.LayoutParams.FLAG_DRAWS_SYSTEM_BAR_BACKGROUNDS);
            //设置状态栏颜色
            window.setStatusBarColor(getResources().getColor(R.color.gold));
            //设置导航栏颜色
            window.setNavigationBarColor(getResources().getColor(R.color.gold));
        }