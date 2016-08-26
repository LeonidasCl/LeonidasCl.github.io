---
layout: post
title:  "ListView强化和优化的基本法【一】"
date:   2016-07-14 23:11:27
categories: Android
comments: true
---
今天想记录的重点其实是ListView的优化，想要优化ListView首先得明白ListView基本的机制，想要明白ListView的机制就得搞清楚适配器的那几个方法是干什么用的。之前只是十分简略的介绍了一下，现在我再做一个相对简略的介绍：

`getCount`方法：返回list含有的项个数，注意是list，不是listview，这个list就是构造Adapter时传入的数据清单。这个方法先于getview执行，如果getcount返回0了，就说明list中没有数据，最关键的getView也就不会执行。

`getItem`和`getItemId`是获取list对应下标处的单个对象的，在getview中我们可能会调用它们来获取单个表单项的某个数据。

`getview`：坠关键的方法，参数表是(int position, View convertView, ViewGroup parent)，这个方法是系统自动调用，让我们去填充数据，完成内部实现，并返回一个我们填充好数据的view。第一个参数是当前view对应的表单项在list中的下标位置，第二个参数是view缓存，这个缓存后文再提，第三个参数是父view也就是这个子view所在的listview。

在介绍convertView这个参数前必须介绍listview的基本的机制，我们的每个list表单项都是一份子view，这份子view是由我们写好的表单项xml解析出来的，整个listview是一个ViewGroup。listview的表单项可能成千上万，用户的屏幕空间只够显示那么几个，显然不能够把所有的view都创建出来，坠好的情况是，用户的屏幕能最多显示几个view，系统就创建几个view，这样最省内存。那安卓是怎么做的呢，可以参考下面这个表：

| 1.刚创建          |2.向上拉了半个view |3.再向上拉一个view |
|-------------------|:-----------------:| -----------------:|
|ViewA-有数据list[0]|ViewA没有完全隐藏  |ViewB没有完全隐藏  |
|ViewB-有数据list[1]|ViewB-有数据list[1]|ViewC-有数据list[2]|
|ViewC-有数据list[2]|ViewC-有数据list[2]|ViewD-有数据list[3]|
|ViewD-有数据list[3]|ViewD-有数据list[3]|ViewE-有数据list[4]|
|ViewE-有数据list[4]|ViewE-有数据list[4]|ViewF-有数据list[5]|
|ViewF-有数据list[5]|ViewF-有数据list[5]|ViewG-有数据list[6]|
|                   |ViewG没有完全显示  |ViewA没有完全显示  |

状态1时listview刚创建，正好能装下五个view，这时向上拉，在viewA没有完全隐藏时，系统还会再创建一个viewG用来存list[6]的数据。从viewA到viewG，共调用七次adapter的getView方法，这七次调用，传进去的convertView都为null。但是，当我们再向下拉，getView再次被调用时，就会发现convertView就不为null了，这时候的convertView是viewA，且填充好了list[0]的数据，什么回事呢？

系统为了节省开支少创建新的view，在viewA变得不可见后，就将viewA缓存起来，等到要用新的view了，就把A丢到getView里面去给用户填充新数据，实现view的复用。不管用户怎么滚动，view的总数来回就是那么几个，view不可见的时候只要能够复用就会被缓存，然后从convertView里传到getView给用户填充。最开始创建的时候，没有view被缓存，convertView就当然一直是null了。不管list有多少项，哪怕几百万项，view就是那么几个，要显示的时候就重新给旧view填充新的数据，这就是listview基本的机制。

这样代码就很容易写了————就是判断convertView是不是null，如果是就创建新的，设置数据，返回新的；如果不是就直接拿convertView设置好数据后返回，像这样：

**这份代码就是在上一篇文章的基础上改的**

    @Override
    public View getView(int i, View view, ViewGroup viewGroup) {
        if(view ==null){
            layoutInflater = LayoutInflater.from(context);
            view = layoutInflater.inflate(R.layout.item_list, viewGroup, false);
        }

        //省略：给表单项中各组件设置数据

        return view;
    }


这算不算是listview的第一个优化呢？我觉得不算，这只是getView的正确使用方式，按照系统要求的方式去复用view，当然如果不管convertView，每次getView都调用inflate创建新的view出来，程序也是可以正常运行的，然而inflate要解析xml，比较耗时，这种做法时间和空间上都很浪费。

说到解析XML比较费时间，优化的方向就明确了：减少解析xml的次数，那在这个getview里哪些方法需要解析xml？两种，1.inflate方法，2.给组件设置数据的时候需要把组件找出来赋给相应的成员，要用findViewById。inflate方法的调用次数已经通过convertView的复用降到最低了，现在就要来减少findViewById的调用次数。写代码之前先来回顾我们给表单项的view设置数据的步骤：

1.建立对应的对象成员（比如要给TextView设置数据就得建一个TextView成员）
=========================================================================

2.把成员通过findViewById同界面上组件绑定，例如：
=================================================

    textView1=(TextView)convertView.findViewById(R.id.item_textview1);

convertView就是传进来可复用的view，是一个表单项，其中肯定包括了要填充的TextView，直接调用findViewById也能找到，但那样搜索xml节点的范围就又扩大了，这个细节是第二次提了。

3.根据getView传进来的下标i调用getItem拿到单个表单项的数据，然后取出来，设置到TextView里面去：
=============================================================================================

    textView1.setText("xxxxxxxxxxxxxxxxxxxx");

getText是**表单项数据模型**中自己写的获取数据成员的get方法。这个**表单项数据模型**我先前置声明一下。

减少findViewById就得把find的结果，也就是绑定的结果存起来————之前我们只是用list对象存了所有表单项的数据，每次都是重新绑定view和控件。既然已经知道view只有那么几个，那我们可以先把控件的引用封装成类保存成对象存到要绑定的view里，这样可复用的view随着convertView传进来时，绑定好的表单项组件（view里面的TextView什么的）就随着convertView传进来了，我们直接拿到就可以，不需要再findViewById。这是listview的第二个优化：

首先建一个类，我把这种类叫做数据模型类，就是前面提到的**表单项数据模型**，它把一个表单项的数据都封装了起来：

    public class Listitem {
    
        private String title;
        private String subtitle;
        private String content;
        private Drawable avatar;
        private String mainbgUrl;
        private Bitmap mainbg;
    
        //所有成员都写上get和set方法，此处省略
    }

然后，之前不是说要存放find的结果吗，所以要搞一个类把这些搜到的view存起来，这个类我们按其功能命名，叫ViewHolder，这个类就放在用到该ListView的Activity内部了，找起来方便:

    public class ViewHolder{

        TextView title;
        TextView subtitle;
        TextView content;
        ImageView mainbg;

        public void setViewHolder(View view) {
            mainbg=(ImageView)view.findViewById(R.id.mainbg);
            title = (TextView) view.findViewById(R.id.title);
            subtitle = (TextView) view.findViewById(R.id.subtitle);
            content = (TextView) view.findViewById(R.id.content);
        }
    }

然后把Adapter也放到Activity内部（实际情况不推荐，这里放在一起便于阅读），注意对Adapter的修改：添加了两个成员变量，getView有较大修改，添加一个getItemView方法专用于填充数据，当然getItem返回类型也改了

    public class ListviewAdapter extends BaseAdapter {

        private Context context;
        private ViewHolder viewHolder;
        private List<Listitem> list;

        public ListviewAdapter(Context ctx,List<Listitem> lst){
            context=ctx;
            list=lst;
        }

        @Override
        public int getCount() {
            return 10;
        }

        @Override
        public Listitem getItem(int i) {
            return list.get(i);
        }

        @Override
        public long getItemId(int i) {
            return i;
        }

        protected void getItemView(View view, ViewHolder viewHolder,final Listitem item) {
            final ViewHolder holder=viewHolder;
            holder.title.setText(item.getTitle());
            holder.subtitle.setText(item.getSubtitle());
            holder.content.setText(item.getContent());
            holder.mainbg.setImageDrawable(getResources().getDrawable(R.drawable.mainbg));
        }

        @Override
        public View getView(int i, View view, ViewGroup viewGroup){

            if (view==null){
                LayoutInflater layoutInflater=LayoutInflater.from(context);
                view=layoutInflater.inflate(R.layout.item_list,viewGroup,false);
                viewHolder = getViewHolder();//每次view是null时都创建一个viewHolder对象，能保证view和holder一一绑定
                viewHolder.setViewHolder(view);//执行一次性的搜索并绑定控件对象到Holder

                getItemView(view, viewHolder,getItem(i));//在这里填充数据

                view.setTag(viewHolder);//以设置标签的方式绑定viewHolder到这个view
            }

            viewHolder= (ViewHolder)view.getTag();//如果这个view存在，那么肯定绑定了Holder，我们就去获取
            getItemView(view,viewHolder,getItem(i));//获取到之后重新设置数据

            return view;
        }

        protected ViewHolder getViewHolder() {
            return new ViewHolder();
        }
    }

再把Activity里自己写的init()函数准备好，用来初始化数据的，实际开发时可在此获取网络数据

    private List<Listitem> init() {
        list=new ArrayList();
        for(int i=0;i<10;i++){
            Listitem item=new Listitem();
            item.setTitle("title " + (i + 1));
            item.setSubtitle("subtitlesubtitle");
            item.setContent("Content Content Content Content Content");
            /*图片可以从网络获取，就在这获取
            item.setMainbg(BitmapFactory.decodeResource(getResources(),R.drawable.mainbg));*/
            list.add(item);
        }
        return list;
    }

最后看看Activity的onCreate里的这三句话，应该是这个样子：

        listview=(ListView)findViewById(R.id.listview);
        ListviewAdapter adapter2=new ListviewAdapter(this,init());
        listview.setAdapter(adapter2);

主要实现几个关键点：1.用view自带的setTag和getTag设置和获取tag，把viewholder存进或取出。2.绑定操作只在建立新的viewholder后执行一次，可以放在构造方法里，不过我不推荐，个人感觉那样不利于解耦3.获得holder后直接获取holder的成员，然后设置数据。

运行一下，可以打断点在getView里，最后会发现view确实实现了复用。

经过前面的优化这个listview已经是具备了满足一般用户需求的能力了，但是对于一些粗暴的用户还可能会出问题。设想：某个用户可能会一直快速上拉，导致listview一直在快速滚动出后面的条目，需要加载很多数据，这些数据里包含媒体文件，甚至是流媒体文件，而这个用户的网速又没有那么快，那会出现什么情况？实践证明这样做的时候，真正在用户屏幕上显示的view会因为之前的view还没有加载完数据而加载缓慢，用户看到的将是假数据，而不是想要的内容，那这个怎么办？只能下次再想办法咯。