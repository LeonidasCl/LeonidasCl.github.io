---
layout: post
title:  "ListView强化和优化的基本法【二】"
date:   2016-07-17 17:46:45
categories: Android
comments: true
---

现在最简单的ListView大家都会，实现起来只需要做三件事：

    1.建一个list，把它丢到界面布局的ListView里面去

    2.建一个Adapter（适配器）

    3.调用ListView的setAdapter把适配器设置了

这样就做好了一个listview。实现适配器这个步骤最关键，写法大概是这个样子：

1.建立对应的对象成员（比如要给TextView设置数据就得建一个TextView成员）

2.把成员通过findViewById同界面上组件绑定，例如：

`textView1=(TextView)convertView.findViewById(R.id.item_textview1);`

convertView就是传进来可复用的view，是一个表单项，其中肯定包括了要填充的TextView，直接调用findViewById也能找到，但那样搜索xml节点的范围就又扩大了，这个细意
3.根据getView传进来的下标i调用getItem拿到单个表单项的数据，然后取出来，设置到TextView里

`textView1.setText(getItem(i).getText()

getText是表单项数据模型中自己写的获取数据成员的get

当然，通常情况下这么简单的listview是不能满足需求的，通常我们要根据用户的需要，单个表单项的布局会变得复杂，比如包含图片啊视频啊，搞个评论点个赞啊，弹个小窗对ListView的展示方式也可能多样化，刷新啊，无限加载啊，都还是有点复杂的，我暂且把增加这些功能叫做listview的强化了，那么怎么来搞这个强化呢？

首先要实现下拉刷新和上拉加载肯定得监听滑动吧；再有，下拉的时候listview被拉下来了，得有个头部布局去填那块拉下来的空白，提示用户下拉刷新或者正在刷新，这个完得隐藏，一开始也得隐藏，同时上拉的时候也得有一个底部布局提示用户正在加载更多信息。已经明确三个基本需求了，所以我就写了一个类，继承ListView并实现AbsListVOnScrollListener，这个监听器是坠吼的，因为到时候用监听来控制加载数据，三个view（顶部布局、listview、底部布局）的显示用回调机制来写，既明确又不冲突。

代码很多所以只贴关键部位了，头部view加上去因为一开始要隐藏，所以得设置整个view的padding为自身的高度负值，padding这么玩也是很有套路的


    private void initHeaderView() {
        headerView = View.inflate(getApplicationContext(), R.layout.refresh_header, nul
        //省略：用findViewById找到各子view，这些子view和header都是listview
        headerView.measure(0, 0); //直接获取高度会获取到0，只能用这种办法量出header的实际高度
        headerViewHeight = headerView.getMeasuredHeight();
        headerView.setPadding(0, -headerViewHeight, 0, 0);//padding要设置好，一开始要隐藏
        this.addHeaderView(headerView); //这个方法是View类原有的，创建好我们的header将其传进去就可以了
        initAnimation();
    }


底部view的初始化也一个套路，这两个方法的调用我放在构造方法里了，让头部和底部view和listview一起创建。

接下来重写onTouchEvent方法，在这里实现下拉的时候把header调出来，如果拉到一定距离就调用刷新方法，header完全显示了要提示不用再拉了，松开刷新（像QQ那样），候判断是不是到底了，到了就调用加载更多数据的方法。这个实现需要调节很久很麻烦，大概就是用传进来的MotionEvent对象调用getY去获取下滑距离，在MotionEvACTION_DOWN和MotionEvent.ACTION_MOVE中去getY分别可以获得按下时候和滑动中的Y坐标，做个差可以得下滑距离。之前那个headerview已经是本类成员了，在这里算完之接setPadding实现下拉显示的各种。总之是很麻烦很麻烦，得慢

在onTouchEvent里判断可以下拉刷新的时候就调用下拉刷新方法。下拉和上拉两个加载的方法可以统一放在一个接口里，让ListView持有这个接口的引用作为成员，在ListVie上拉下拉刷新的方法。然后，让要使用ListView的类，例如Activity中，去实现那个统一的接口，把刷新加载数据填进去。这也算是一种自制的回调机制吧。

我们实现了滚动监听接口，所以要写上onScrollStateChanged和onScroll，onScroll可以判断是否滑到底部了，在这里把底部的flag立起，先不要调用对应方法，等进了onScrolleChanged再去收这个flag，根据flag状态来决定是否要调用加载更多数据。