---
layout: post
title:  "ListView强化和优化的基本法【二】"
date:   2016-07-17 23:11:27
categories: Android
comments: true
---

    今天想记录的重点其实是ListView的优化，想要优化ListView首先得明白ListView基本的机制，想要明白ListView的机制就得搞清楚适配器的那几个方法是干什么用的，我做了一个大概的总结：

    getCount方法：返回list含有的项个数，注意是list，不是listview，这个list是构造Adapter时传入的数据清单。这个方法先于getview执行，如果getcount返回0了，就说明list中没有数据，最关键的getView也就不会执行。

    getItem和getItemId是获取list对应下标处的单个对象的，在getview中会用到他们来获取单个表单项的数据。

    getview：坠关键的方法，(int position, View convertView, ViewGroup parent)，这个方法是系统自动调用，让我们完成内部实现，并返回一个我们填充好数据的view。第一个参数是当前view对应的表单项在list中的下标位置，第二个参数是view缓存，这个缓存后文再提，第三个参数是父view也就是这个子view所在的listview。

    在介绍convertView这个参数前必须介绍listview的基本的机制，我们的每个list表单项都是一份子view，这份子view是由我们写好的表单项xml解析出来的，整个listview是一个ViewGroup。listview的表单项可能成千上万，用户的屏幕空间只够显示那么几个，显然不能够把所有的view都创建出来，坠好的情况是，用户的屏幕能最多显示几个view，系统就创建几个view，这样最省内存。那安卓是怎么做的呢，可以参考下面这个表：

    | 1.刚创建          |2.向上拉了半个view |3.再向上拉一个view |
    |-------------------|:-----------------:| -----------------:|
    |ViewA-存数据list[0]|ViewA没有完全隐藏  |ViewB没有完全隐藏  |
    |ViewB-存数据list[1]|ViewB-存数据list[1]|ViewC-存数据list[2]|
    |ViewC-存数据list[2]|ViewC-存数据list[2]|ViewD-存数据list[3]|
    |ViewD-存数据list[3]|ViewD-存数据list[3]|ViewE-存数据list[4]|
    |ViewE-存数据list[4]|ViewE-存数据list[4]|ViewF-存数据list[5]|
    |ViewF-存数据list[5]|ViewF-存数据list[5]|ViewG-存数据list[6]|
    |                   |ViewG没有完全显示  |ViewA没有完全显示  |

    状态1时listview刚创建，正好能装下五个view，这时向上拉，在viewA没有完全隐藏时，系统还会再创建一个viewG用来存list[6]的数据。从viewA到viewG，共调用七次adapter的getView方法，这七次调用，传进去的convertView都为null。但是，当我们再向下拉，getView再次被调用时，就会发现convertView就不为null了，这时候的convertView是viewA，且填充好了list[0]的数据，什么回事呢？

    系统为了节省开支少创建新的view，在viewA变得不可见后，就将viewA缓存起来，等到要用新的view了，就把A丢到getView里面去给用户填充新数据，实现view的复用。不管用户怎么滚动，view的总数来回就是那么几个，view不可见的时候只要能够复用就会被缓存，然后从convertView里传到getView给用户填充。最开始创建的时候，没有view被缓存，convertView就当然一直是null了。不管list有多少项，哪怕几百万项，view就是那么几个，要显示的时候就重新给旧view填充新的数据，这就是listview基本的机制。

    这样代码就很容易写了————就是判断convertView是不是null，如果是就创建新的，设置数据，返回新的；如果不是就直接拿convertView设置好数据后返回，像这样：

    `
    @Override
    public View getView(int i, View view, ViewGroup viewGroup) {
        if(view ==null){
            layoutInflater = LayoutInflater.from(context);
            view = layoutInflater.inflate(R.layout.item_list, viewGroup, false);
            ////省略：给表单项中各组件设置数据
            return view;
        }

        //省略：给表单项中各组件设置数据

        return view;
    }
    `

    这算不算是listview的第一个优化呢？我觉得不算，这只是getView的正确使用方式，按照系统要求的方式去复用view，当然如果不管convertView，每次getView都调用inflate创建新的view出来，程序也是可以正常运行的，然而inflate要解析xml，比较耗时，这种做法时间和空间上都很浪费。

    说到解析XML比较费时间，优化的方向就明确了：减少解析xml的次数，那在这个getview里哪些方法需要解析xml？两种，1.inflate方法，2.给组件设置数据的时候需要把组件找出来赋给相应的成员，要用findViewById。inflate方法的调用次数已经通过convertView的复用降到最低了，现在就要来减少findViewById的调用次数。写代码之前先来回顾我们给表单项的view设置数据的步骤：

    1.建立对应的对象成员（比如要给TextView设置数据就得建一个TextView成员）

    2.把成员通过findViewById同界面上组件绑定，例如：

    `textView1=(TextView)convertView.findViewById(R.id.item_textview1);`

    convertView就是传进来可复用的view，是一个表单项，其中肯定包括了要填充的TextView，直接调用findViewById也能找到，但那样搜索xml节点的范围就又扩大了，这个细节也要注意一下。

    3.根据getView传进来的下标i调用getItem拿到单个表单项的数据，然后取出来，设置到TextView里面去：

    `textView1.setText(getItem(i).getText());`

    getText是表单项数据模型中自己写的获取数据成员的get方法。

    减少findViewById就得把find的结果，也就是绑定的结果存起来————之前我们只是用list对象存了所有表单项的数据，每次都是重新绑定view和控件。既然已经知道view只有那么几个，那我们可以先把控件的引用封装成类保存成对象存到要绑定的view里，这样可复用的view随着convertView传进来时，绑定好的表单项组件（view里面的TextView什么的）就随着convertView传进来了，我们直接拿到就可以，不需要再findViewById。这是listview的第二个优化：

    `//adapter基类中
    @Override
    public View getView(int i, View view, ViewGroup viewGroup) {
        if(view ==null){
            layoutInflater = LayoutInflater.from(context);
            view = layoutInflater.inflate(R.layout.item_list, viewGroup, false);
            viewHolder = getViewHolder();
            viewHolder.setViewHolder(view);
            getItemView(view, viewHolder,getItem(i));
            view.setTag(viewHolder);
            return view;
        }

        viewHolder = (ViewHolder)view.getTag();
        getItemView(view,viewHolder,getItem(i));

        return view;
    }
    `

    `//adapter实现类中，这个实现类在用到listview的activity内部
        @Override
        protected void getItemView(View view, ViewHolder viewHolder,final ListItemA item) {
            final ViewHolderItemA holder=(ViewHolderItemA)viewHolder;
            holder.title.setText(item.getTitle());
            holder.subtitle.setText(item.getSubtitle());
            holder.content.setText(item.getContent());

            }

        //用这个方法创建新的viewholder
        @Override
        protected ViewHolder getViewHolder() {
            return new ViewHolderItemA();
        }
        
    `
    `//viewholder实现，这个实现类同样在用到listview的activity内部
    public class ViewHolderItemA implements ViewHolder{

        TextView title;
        TextView subtitle;
        TextView content;
        ImageView mainbg;

        @Override
        public void setViewHolder(View view) {
            mainbg=(ImageView)view.findViewById(R.id.mainbg);
            title = (TextView) view.findViewById(R.id.title);
            subtitle = (TextView) view.findViewById(R.id.subtitle);
            content = (TextView) view.findViewById(R.id.content);
        }
    }
    `

    `//viewholder接口
        public interface ViewHolder {
            public abstract void setViewHolder(View view);
        }
    `

    主要实现几个关键点：1.用view自带的setTag和getTag设置和获取tag，把viewholder存进或取出。2.绑定操作只在建立新的viewholder后执行一次，可以放在构造方法里，不过我不推荐，个人感觉那样不利于解耦，所以加了个方法用来绑定并写成接口。3.获得holder后直接获取holder的成员，然后设置数据。

    经过前面的优化这个listview已经是具备了满足一般用户需求的能力了，但是对于一些粗暴的用户还可能会出问题。设想：某个用户可能会一直快速上拉，导致listview一直在快速滚动出后面的条目，需要加载很多数据，这些数据里包含媒体文件，甚至是流媒体文件，而这个用户的网速又没有那么快，那会出现什么情况？实践证明这样做的时候，真正在用户屏幕上显示的view会因为之前的view还没有加载完数据而加载缓慢，用户看到的将是假数据，而不是想要的内容，那这个怎么办？只能下次再想办法咯。