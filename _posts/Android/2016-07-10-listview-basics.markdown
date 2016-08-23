---
layout: post
title:  "ListView的基本法"
date:   2016-07-10 10:33:07
categories: Android
comments: true
---


ListView是安卓界面中必不可少的组件，今天直入正题介绍ListView及其用法。

我们通常所说的ListView其实只是一个容器，我们要做的就是给这个容器中填充list表单项，填充的内容就能在容器上显示。这个填充的过程通过调用ListView的setAdapter()方法来实现。这个方法接受的参数是一个ListAdapter对象，ListAdapter呢是一个接口。所以只要实现了这个接口的类就能放到setAdapter()的参数中去。在安卓中各种Adapter的类图是这样的：

![图片](http://obdvl7z18.bkt.clouddn.com/image/20160710/01.png)

可以看到BaseAdapter的所有子类都实现了ListAdapter，我们对ListView来进行填充主要就是用的ArrayAdapter和SimpleAdapter。ArrayAdapter非常简单好用，但是只能填充TextView；SimpleAdapter可以填充各种控件。但如果我们想取得完全的控制权，那最好还是继承BaseAdapter写一个自己的Adapter。

首先从最简单的开始，做一个ArrayAdapter显示的ListView：

在Activity的布局中添加一个ListView：

![图片](http://obdvl7z18.bkt.clouddn.com/image/20160710/02.jpg)

在创建一个布局xml文件，作为单个表单项的布局，只能有TextView，这里为了方便只写了一个TextView，写多个也是可以的

![图片](http://obdvl7z18.bkt.clouddn.com/image/20160710/03.jpg)

在该Activity的onCreate中，主要写四行代码，只是一点点微小的工作：

![图片](http://obdvl7z18.bkt.clouddn.com/image/20160710/04.jpg)

然后运行我们的app，就能看到效果了：

![图片](http://obdvl7z18.bkt.clouddn.com/image/20160710/05.jpg)

接下来再试试SimpleAdapter，用这个Adapter我们就可以处理较为复杂的界面了。首先还是要在对应的Activity中添加一个ListView（ListView就像个容器，当然也可以用原来的，不过为了防止混淆还是新写了一个）：

![图片](http://obdvl7z18.bkt.clouddn.com/image/20160710/06.jpg)

然后还是要写一个布局xml文件，作为表单项，这界面就给一个代码：

    <?xml version="1.0" encoding="utf-8"?>
    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
                  android:orientation="horizontal"
                  android:layout_width="match_parent"
                  android:layout_height="match_parent">
        <ImageView
            android:id="@+id/avatar"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:src="@drawable/avatar"
            android:layout_margin="10dp"/>
    
        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_margin="10dp"
            android:orientation="vertical">
    
        <TextView
            android:id="@+id/title"
            android:layout_width="wrap_content"
            android:layout_height="match_parent"
            android:textSize="16sp"
            android:text="@string/title"/>
    
        <TextView
            android:id="@+id/subtitle"
            android:textSize="16sp"
            android:layout_width="wrap_content"
            android:layout_height="match_parent"
            android:text="@string/subtitle"/>
    
            <TextView
                android:id="@+id/content"
                android:textSize="16sp"
                android:paddingTop="20dp"
                android:layout_width="wrap_content"
                android:layout_height="match_parent"
                android:text="@string/content"/>
    
            <ImageView
                android:id="@+id/mainbg"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:adjustViewBounds="true"
                android:scaleType="fitCenter"
                android:src="@drawable/mainbg"
                android:contentDescription="@string/app_name"/>
    
    </LinearLayout>
    </LinearLayout>

接下来步骤还是要在Activity的onCreate()中，写上那么三句话。

![图片](http://obdvl7z18.bkt.clouddn.com/image/20160710/07.jpg)

这里说明一下，SimpleAdapter()的最后两个参数都是数组，它们描述了数据的格式：最后一个参数是int数组，元素是list表单项布局里的单个控件的id；倒数第二个参数是string数组，是这些控件对应的要填充的数据的键值名，这个键值名就是init()方法里返回的单个表单项中的键值名，通过它索引到我们添加好的数据。init()方法是自己写的，用来初始化数据，返回一个list，这个list的每一项都是一个Map，每一个Map就包含了单个表单项的所有数据，看到这个Map的键、值类型和SimpleAdapter()后两个参数的单项的类型相容，就能够理解了。

    private List<Map<String,Object>> init() {
        List<Map<String,Object>> lst=new ArrayList<>();
        for (int i=0;i<10;i++){
            Map<String,Object> item=new HashMap<>();
            item.put("itemAvatar",R.drawable.avatar);
            item.put("itemContent","content content content content"+i);
            item.put("itemTitle","title"+i);
            item.put("itemSubtitle","subtitle subtitle"+i);
            lst.add(item);
        }
        return lst;
    }

然后还是要运行一下下：

![图片](http://obdvl7z18.bkt.clouddn.com/image/20160710/08.jpeg)

到现在为止，这两种毫不费力的Adapter创建方式可以说是掌握了。这两种方式可以说是很便捷，但是应对复杂需求时显得吃力。比如说我要在适当的时候从网络加载图片，或者要从缓存中取数据等等。这种时候最后的办法就是继承BaseAdapter搞一个自己的Adapter类，原来不是在构造方法里填好数据和控件，让系统去自动填充吗，现在我们自己去填充。搞的这个自己的Adapter类代码在这：

    public class ListviewAdapter extends BaseAdapter {
    
        private Context context;
    
        public ListviewAdapter(Context ctx){
            context=ctx;
        }
    
        @Override
        public int getCount() {
            return 10;
        }
    
        @Override
        public Object getItem(int i) {
            return null;
        }
    
        @Override
        public long getItemId(int i) {
            return i;
        }
    
        @Override
        public View getView(int i, View view, ViewGroup viewGroup){
    
            LayoutInflater layoutInflater=LayoutInflater.from(context);
            view=layoutInflater.inflate(R.layout.item_list,viewGroup,false);
            ImageView avatar=(ImageView)view.findViewById(R.id.avatar);
            avatar.setImageDrawable(avatar.getResources().getDrawable(R.drawable.avatar));
            ImageView mainbg=(ImageView)view.findViewById(R.id.mainbg);
            mainbg.setImageDrawable(avatar.getResources().getDrawable(R.drawable.mainbg));
            TextView subtitle=(TextView)view.findViewById(R.id.subtitle);
            subtitle.setText("subtitle subtitle");
            TextView title=(TextView)view.findViewById(R.id.title);
            title.setText("title");
            TextView content=(TextView)view.findViewById(R.id.content);
            content.setText("content content content content");
    
            return view;
        }
    }

可以看到我们继承BaseAdapter要实现四个方法，除了getView之外的三个方法现在暂时不需要用到，以后优化的时候再讲，这篇文章先粗略的介绍一下getView()，这个方法就是让我们填充view的，返回的就是填充好的view。我们先用LayoutInflater去渲染一个view出来，这个view是我们用表单项xml创建的，自然会包含那些要填充的控件，所以我们就调用这个view的findViewById方法去寻找。当然也可以直接调用findViewById去寻找，但是搜索的范围会变大，性能就降低了很多，这个方法通过搜索xml节点去寻找元素，相对来说比较耗时。找到各控件后就给它们填充数据，如果要从网络获取，也在这里发起网络请求。处理完这一些列控件后就把view返回去，这个view就可以显示在屏幕上了。现在先不要问getView方法传进来的那三个参数怎么用，下回再慢慢分析。最后放一张自定义Adapter后的效果图：

![图片](http://obdvl7z18.bkt.clouddn.com/image/20160710/09.jpeg)