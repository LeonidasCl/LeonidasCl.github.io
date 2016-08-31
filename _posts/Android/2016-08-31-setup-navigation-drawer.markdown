---
layout: post
title:  "深入理解Activity【二】为Activity手动加上侧边栏"
date:   2016-08-31 00:22:17
categories: Android
comments: true
---



​	新建Activity的时候如果使用向导选择安卓自带的，我们会见到一种有侧滑菜单的Activity，就是NavigationDrawerActivity。

​	创建NavigationDrawerActivity后我们可以通过滑动或是点击左上角actionbar的按钮去打开一个侧滑面板，实现部分界面的快速访问，通常用来做为展示用户信息的控件。不过随着安卓版本的更新，对于actionBar谷歌已经不再推荐使用（推荐用Toolbar代替），侧滑栏的设计也被越来越多的app放弃了。然而有时候就是有这样的需求，要去实现这个侧滑栏，这时候我们就不好再创建安卓自带的NavigationDrawerActivity了，最好手动添加这个侧滑菜单，以取得最大控制权。

​	手动添加有很多种方式，这里讲比较简便的一种。首先注意我们的布局文件，要想让一个Activity支持侧滑菜单，我们要对其布局文件进行相应的设置，满足各种条件。首先布局文件的根布局必须是DrawerLayout，在这个根布局下，最好只有两个子布局——第一个是主界面，第二个是侧滑菜单的布局。需要注意的是，这个主界面的宽和高要和根布局也就是DrawerLayout一样才行。对于侧滑菜单布局的要求则主要有两点：为了让其能在所有设备上显示，宽度不能超过320dp；必须设置android:layout_gravity参数。

​	清楚要求之后我们就写一个简单的布局，其实我直接把安卓自带的NavigationDrawerActivity里的侧滑菜单布局复制过来了：

```java
<android.support.v4.widget.DrawerLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/main_drawer_layout"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:fitsSystemWindows="true"
    tools:context=".Activity.MainActivity">
<!--这个布局是主页面的布局-->
<android.support.design.widget.CoordinatorLayout
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:layout_marginTop="@dimen/nav_header_vertical_spacing">
    <include
        layout="@layout/toolbar_layout"
        android:layout_width="match_parent"
        android:layout_height="?attr/actionBarSize"
        android:gravity="center"/>
    <include layout="@layout/content_main" />
</android.support.design.widget.CoordinatorLayout>
<!--这个布局是侧滑菜单的布局-->
    <android.support.design.widget.NavigationView
        android:id="@+id/nav_view"
        android:layout_width="250dp"
        android:layout_height="match_parent"
        android:layout_gravity="start"
        android:fitsSystemWindows="true"
        app:headerLayout="@layout/nav_header_main"
        app:menu="@menu/activity_main_drawer"/>

</android.support.v4.widget.DrawerLayout>
```

​	侧滑菜单布局里面用到的资源全是生成NavigationDrawerActivity的时候自带的（为了偷懒）。

​	布局创建完成之后，我们在Activity里找到这个控件，并设置监听。首先让这个Activity实现NavigationView.OnNavigationItemSelectedListener，之后会让我们实现onNavigationItemSelected方法，在这个方法中我们去处理点击事件。

先看Activity的onCreate方法：

```java
        //这个是父布局，也就是这个Activity的根布局
        mDrawerLayout = (DrawerLayout) findViewById(R.id.main_drawer_layout);
        //找到侧边栏
        navigationView = (NavigationView) findViewById(R.id.nav_view);
        //为侧边栏设置监听，由于此Activity已实现OnNavigationItemSelectedListener接口，可以传this
        navigationView.setNavigationItemSelectedListener(this);
        //这是左上角一个普通的按钮，除了滑动还可以点击它来打开侧滑菜单
        btnAvartar=(ImageButton)findViewById(R.id.toolbar_btn_avatar);
        btnAvartar.setOnClickListener(new View.OnClickListener() {
                @Override
                public void onClick(View v) {
                    if (isLogin){
                        //如果用户已经登录，打开侧滑菜单
                        mDrawerLayout.openDrawer(navigationView);
                    }else {
                        //如果没有登录，跳转到注册
                        Intent intent = new Intent("android.intent.action.LOGINACTIVITY");
                        intent.putExtra("method", StatusCode.STATUS_LOGIN);
                        startActivity(intent);
                    }

                }
            });
```

再看看我们的监听器：

```
   @Override
    public boolean onNavigationItemSelected(MenuItem item) {
        int id = item.getItemId();

        if (id == R.id.nav_camera) {
        } else if (id == R.id.nav_gallery) {
        } else if (id == R.id.nav_slideshow) {
        } else if (id == R.id.nav_manage) {
        } else if (id == R.id.nav_share) {} 
        else if (id == R.id.nav_send) {}
        
        DrawerLayout drawer = (DrawerLayout) findViewById(R.id.main_drawer_layout);
        drawer.closeDrawer(GravityCompat.START);
        return true;
    }
```

​	从上面的代码可以知道，这次十分偷懒，连监听方法都复制的安卓自带NavigationDrawerActivity里面的。其实我们的侧滑菜单不仅限于NavigationView，可以是任何布局，比如一个list，一张表，几幅图什么的。只要我们满足前置的几个条件，我们的Activity就能顺利使用侧滑菜单。现在再总结一下：

1.根布局是DrawerLayout，下面一级只有两个子控件，两个子控件和父控件的大小关系满足上文要求。

2.为侧滑菜单设置监听（侧滑菜单控件是什么就设置什么监听，这次例子是NavigationView，设置的NavigationView.OnNavigationItemSelectedListener，如果是个ListView，就要设置ListView.OnItemClickListener了）。

所以这没有什么神奇的东西，就是很普通的一个布局，找到父控件也就是DrawerLayout后调用其closeDrawer和openDrawer就可以随意打开和关闭侧滑菜单，不复杂。