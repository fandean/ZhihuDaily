# ZhihuDaily项目笔记

## 问题汇总

- 检测List是否为空，每次检测都不为null，（该list中嵌套了一个子list）
因为该list中每次都使用add()添加了一个由new ArrayList<>()创建的子list。
- 将ZhihuNews对象转换为Json然后存入数据库（因为bean没有设计好）时，不知是由于json字符串太长，
还是因为json中存在特殊字符？
之后采用如下方法进行转换，想以数据库中的BLOB，但还是不行，获取数据时没有反应。（由于手机没有root，不知道是否插入了数据）
  ```
          String json = gson.toJson(zhihuNews);
          //String -> byte[]
          byte[] jsonBytes = json.getBytes(Charset.forName("UTF-8"));
          //byte[] -> String
          String str = new String(jsonBytes,Charset.forName("UTF-8"));
  ```
- 缓存问题。头疼。
- 定义field变量， `private SQLiteDatabase mdb = new MyBaseHelper(this).getWritableDatabase();`并且直接初始化。
程序异常退出，因为此时传入的this（即当前Activity）还未创建。
- 从豆瓣列表中打开的Activity中返回到知乎日报列表后知乎Fragment中的列表变空，并且刷新失效，
但可以通过按返回键退出程序，再次进入时可以重新显示知乎列表。
- 切换为横屏后，同样知乎Fragment中的列表变空，而豆瓣却可以显示。
- 对知乎日报的API分析的不够透彻；原来当天的信息量会慢慢增长的，也就是每次更新后获取到的当天的list中节点数是会增加的。
并且详情页中的内容都会改变。
- 判断数据表查询返回的游标中是否有数据和使用何种形式的代码来确保游标的关闭。
- RecyclerView中的包含CardView的item的点击事件的处理。
- RecyclerView的深入学习。

- TabLayout切换监听
- WebView状态的保存和恢复：[Android Fragment使用(三) Activity, Fragment, WebView的状态保存和恢复](http://www.cnblogs.com/mengdd/p/5582244.html "Android Fragment使用(三) Activity, Fragment, WebView的状态保存和恢复 - 圣骑士wind - 博客园")



viewpager.setOnPageChangeListener()，从而使得在你应用中如果想监听ViewPager的页面状态改变

RecyclerView设置分隔线：可以给Item的布局去设置margin，当然了这种方式不够优雅，
我们文章开始说了，我们可以自由的去定制它，当然我们的分割线也是可以定制的。




## FloatingActionButton

|||
| --------- | ---------------------------- |
|Position位置	|You can position the floating button by using `layout_gravity` attribute；还可添加`layout_anchor`来将锚定到你想要的View上 |
|Size大小	|FAB supports two sizes ‘normal‘ and ‘mini‘. You can define the size of the button by using `app:fabSize` attribute |
|Background Color颜色	|By default, fab takes `colorAccent` as background color（. If you want to change the background of fab, use `app:backgroundTint` attribute to define your own background color |
| 涟漪（ripple）交互效果 |`android:clickable="true"` 当你使用了backgroundTint属性来改变系统的背景红色，一定要记得设置clickable属性为true，否则也是无法产生涟漪（ripple）交互效果的|
|设置边距 | 底部推荐为 16dp `android:layout_margin="16dp"`|

[Floating Action Buttons](https://guides.codepath.com/android/floating-action-buttons "Floating Action Buttons | CodePath Android Cliffnotes")
FloatingActionButtons在滑动时的隐藏效果，需要进行自定义。

Background Color颜色,不建议直接使用一般的颜色值，这个属性的值是一个ColorStateList类型 [ColorStateList官网介绍](https://developer.android.com/reference/android/content/res/ColorStateList.html "官网介绍")）


## RecyclerView

[深入浅出 RecyclerView|开源实验室-张涛](https://www.kymjs.com/code/2016/07/10/01/ "深入浅出 RecyclerView|开源实验室-张涛")




通知在列表的末尾添加了插入了多个项目：
```
int curSize = adapter.getItemCount();
//新的items位于 newItems这个链表中
contacts.addAll(newItems);
//通知该范围内数据已经变更
adapter.notifyItemRangeInserted(curSize, newItems.size());
```

通知item数据变更不会滚动到相应位置，可以调用RecyclerView的 scrollToPosition() 进行滚动

> 排序现有列表，见codepath的介绍

### 自定义分割线

使用Android自带的分割线：
```
//对于垂直列表可以使用如下DividerItemDecoration
mRecyclerView.addItemDecoration(new DividerItemDecoration(getActivity(),
                DividerItemDecoration.VERTICAL));
//对于水平列表可以使用
mRecyclerView.addItemDecoration(new DividerItemDecoration(getActivity(),
                DividerItemDecoration.HORIZONTAL));
```

CodePath中介绍了一个源代码示例，该示例在鸿祥的[Android RecyclerView 使用完全解析 体验艺术般的控件](http://blog.csdn.net/lmj623565791/article/details/45059587 "Android RecyclerView 使用完全解析 体验艺术般的控件 - Hongyang - 博客频道 - CSDN.NET")
处有讲解。


### RecycleView无限滚动（上拉加载）

实现滚动所需信息： **列表中最后一个可见Item** 和 **阀值**（以便在到达最后一个item之前开始获取更多数据）

![图片](http://imgur.com/6E7X1pr.png "")


问题：

In order for the pagination system to continue working reliably, you should make sure to clear the adapter of items (or notify adapter after clearing the array) before appending new items to the list.

如果要清除数据，则必须保证在清除工作在追加数据之前完成。

> 比如在选择观看某天的日报时，如果此时列表中已经加载了很多天的的项目，清除工作需要很久才能完成。这时可能会出现问题。还没有发现。

> 我的一个问题是，在选择了日期后，... 无法上拉加载更多。而重启app后又可以加载。后来又没有重现该问题。

上拉加载的日期参考值是当天，导致重选日期后加载的数据是从当天的第二天开始。




## RecyclerView手动移除列表项

[Android RecyclerView, Android CardView Example Tutorial - JournalDev](http://www.journaldev.com/10024/android-recyclerview-android-cardview-example-tutorial "Android RecyclerView, Android CardView Example Tutorial - JournalDev")



### RecycleView其他

- 为Item按下时显示一个“选择”的效果，
我们可以item的布局文件中为根布局设置`android:background`的属性值为`?android:attr/selectableItemBackground`







## SwipeRefreshLayout

[Implementing Pull to Refresh Guide](https://guides.codepath.com/android/Implementing-Pull-to-Refresh-Guide#troubleshooting "Implementing Pull to Refresh Guide | CodePath Android Cliffnotes")

与RecyclerView配合使用，在Adapter中添加如下方法：
```
/* Within the RecyclerView.Adapter class */

// Clean all elements of the recycler
public void clear() {
    items.clear();
    notifyDataSetChanged();
}

// Add a list of items -- change to type used
public void addAll(List<Tweet> list) {
    items.addAll(list);
    notifyDataSetChanged();
}
```


Troubleshooting 故障排查：

- Activity的setContentView()必须在onCreate()方法的第二行调用
- 在数据加载完成后（加载到list后）**才**调用setRefreshing()。
- 如果使用`CoordinatorLayout` to manage scrolling，确保将`app:layout_behavior="@string/appbar_scrolling_view_behavior"` 设置给SwipeRefreshLayout而不是它里面的RecyclerView。
- 更新列表之前先清除旧的items。



进阶：

[hanks-zyh/SwipeRefreshLayout: 谷歌的下拉刷新，新的界面效果](https://github.com/hanks-zyh/SwipeRefreshLayout "hanks-zyh/SwipeRefreshLayout: 谷歌的下拉刷新，新的界面效果")



```
// 设置下拉圆圈上的颜色，蓝色、绿色、橙色、红色
mySwipeRefreshLayout.setColorSchemeResources(
    android.R.color.holo_blue_bright,
    android.R.color.holo_green_light,
    android.R.color.holo_orange_light,
    android.R.color.holo_red_light);

// 通过 setEnabled(false) 禁用下拉刷新
mySwipeRefreshLayout.setEnabled(false);

// 设定下拉圆圈的背景
mSwipeLayout.setProgressBackgroundColor(R.color.red);

```

通过 `setRefreshing(false)` 和 `setRefreshing(true)` 来手动调用刷新的动画。

onRefresh 的回调只有在手势下拉的情况下才会触发，通过 setRefreshing 只能调用刷新的动画是否显示。

SwipeRefreshLayout 也可放在 CoordinatorLayout 内共同处理滑动冲突，有兴趣可以尝试。


前面提到我们自己调用 setRefresh(true) 只能产生动画，而不能回调刷新函数


当SwipeRefreshLayout包含一个子控件和一个子ListView，两个子控件的时候的处理，见：

[Android SwipeRefreshLayout Tutorial](https://www.survivingwithandroid.com/2014/05/android-swiperefreshlayout-tutorial-2.html "Android SwipeRefreshLayout Tutorial")

思路是，对ListView的滚动进行监听，只有当第一个列表项可见的时候才启用SwipeRefreshLayout。


经常见到

```
//在onRefresh()中使用如下方法进行延时。
//延迟处理程序  a post delayed handler
@Override public void onRefresh() {
    new Handler().postDelayed(new Runnable() {
        @Override public void run() {
            swipeLayout.setRefreshing(false);
        }
    }, 5000);
```

```
 int cnt = swiperefresh.getChildCount();
                            for(int i=0; i<cnt; i++) {
                                View ch = swiperefresh.getChildAt(i);
                                if(ch.getClass().getSimpleName().equals("CircleImageView")) {
                                    ch.clearAnimation();
                                    ch.setVisibility(View.GONE);
                                    swiperefresh.setRefreshing(false);
                                    ch.clearAnimation();
                                    break;
                                }
                            }

```


### SwipeRefreshLayout的问题

原因：Refresh()方法无法调用。

排除：子RecyclerView的关系

预测：和协调员CoordinatorLayout有关； 或者和ViewPager有关。

在新建的测试项目中，两个Tab中的SwipeRefreshLayout都无法调用 Refresh()方法；而知乎这个项目只有豆瓣电影那边无法调用。

处理过程：

- 调试程序，调试技术菜，没找出来为什么没有调用Refresh()。
- 查找SwipeRefreshLayout的用法
- 查找是否有类似错误。 没有
- 新建一个项目测试，是否是和CoordinatorLayout有关。新项目复制了部分此项目的代码。无果。
- 修改代码，有带来许多新问题。why？？？
- 绝望之际，根据新情况，难道两个Fragment都这样设置监听器会有冲突？那让它以匿名内部类的形式实现监听器吧！
- 发现问题了，原来我个SB连监听器都没有给它添加进去，叫他怎么调用。




## CardView

TODO CardView的点击效果还没有实现。

由于之前为CardView和Image设置了如下两项，但又没有为它们设置监听器，
所以截获了点击事件，点击无法传递给底部的LineaLayout。
```
android:clickable="true"
android:focusable="true"
```





## 日期的格式化

被一个问题困扰

```
GregorianCalendar todayCalendar = new GregorianCalendar();
todayCalendar.setTimeInMillis(ZhihuNewsLab.get(getActivity()).getToday());
todayCalendar.add(Calendar.DAY_OF_MONTH,-offset);

todayCalendar.get(Calendar.YEAR);
//获取到的月份是没有前导0的，并且是从0开始算起，比如6月它返回的是5，而不是05或6或06。
todayCalendar.get(Calendar.MONTH);
todayCalendar.get(Calendar.DAY_OF_MONTH);
```




## ViewPager

ViewPager的存在原因，《Android权威编程指南》。


RecyclerView与ViewPager的对比：



FragmentStatePagerAdapter 和 FragmentPagerAdapter的区别：
唯一的区别在于，卸载不再需要的fragment时，各自采用的处理方法有所不同。

|FragmentStatePagerAdapter| FragmentPagerAdapter |
|----------------|-----------------|
|会销毁不需要的Fragment|  fragment永远不会被销毁，调用事务的detach()方法来处理它，这样只是销毁Fragment的视图但fragment实例还存在于FragmentManager中 |
|在销毁fragment时可以调用onSaveInstanceState()方法保存信息 | 没有被销毁不必保存 |
| 多个fragment |适用于使用少量固定的fragment|



RecyclerView的Adapter与ViewPager的PagerAdapter的对比：

```
                             {   instantiateItem()
 onBindViewHolder()        --    destroyItem()
                             {   isViewFromObject()
```

isViewFromObject()方法的具体实现： （一行代码）



## Menu菜单

《Android权威编程指南》

### Activity中的菜单

Activity类提供了管理菜单的回到函数。 **需要选项菜单时，**Android会调用Activity的onCreateOptionMenu(Menu)方法。


## toolbar

```
        //替换actionbar，为toolbar，
        //对Toolbar的修改必须要在替换actionbar方法之前, 否则会导致某些属性修改无效, 例如setTitle()标题设置
        setSupportActionBar(mToolbarMain);
```

关于替不替换actionbar有什么区别: 系统默认会对actionbar有一些操作方法,例如默认对actionbar显示了应用名称,
如果执行了替换actionbar方法, 即使不给toolbar设置标题,系统也会默认加个应用名称作为标题


### Toolbar和菜单

分两种情况：

1、当替换了Actionbar

创建菜单文件，重写创建菜单方法；和创建默认菜单方式一样。

2、没有替换Actionbar

`mToolbar.inflateMenu(R.menu.toolbar_inflat_menu);`


如果需要不显示菜单：
可以移除该方法或直接返回false
```
    @Override
    public boolean onCreateOptionsMenu(Menu menu) {
        // Inflate the menu; this adds items to the action bar if it is present.
        getMenuInflater().inflate(R.menu.main, menu);
        //对返回值的描述：You must return true for the menu to be displayed; if you return false it will not be shown.
//        return true;
        //如果需要不显示菜单，可以移除该方法或直接返回false
        return false;
    }
```


### Fragment中的菜单

Fragment有一套自己的选项菜单回调函数。


两个回调方法：

- 创建菜单：  `onCreateOptionsMenu()`
- 响应菜单项选择事件： `onOptionsItemSelected()`

`Fragment.onCreateOptionsMenu()`时由FragmentManager负责调用。

使用该`setHasOptionsMenu(boolean hasMenu)`通知FragmentManager，一般在Fragment的onCreate()方法中调用此方法。

回调函数中的`super.***`方法的作用。



### 实现层级式导航

在manifest文件中的子Activity中添加相关属性。

```
android:parentActivityName=".ui.MainActivity"
```


使用toolbar还需要添加如下代码：`getSupportActionBar().setDisplayHomeAsUpEnabled(true);`才能显示该箭头，
然后它就可以自动响应点击事件了。


> 如果在Fragment中呢？


层级式导航的工作原理：

创建Intent，调用startActivity()，然后finish()当前Activity。

层级导航，重建父Activity的问题。而按返回键不会。


解决办法之一：

取消Manifest文件中指定的父Activity， `android:parentActivityName=".ui.MainActivity"`

然后在代码中使用如下代码：
```
    @Override
    public boolean onOptionsItemSelected(MenuItem item) {

        int id = item.getItemId();
        if (id == android.R.id.home){
            finish();
        }
        return super.onOptionsItemSelected(item);
    }
```





## 对话框

对话框都继承至 Dialog 类，而Dialog类直接继承至Object。它与View类没有继承关系。

Dialog并非通过启动一个新Activity实现。

建议将AlertDialog封装在DialogFragment**实例**中使用(理解这句话)。这样可以解决设备配置变更的问题。


API 21为Activity增加了一个新的属性，只要将其设置成persistAcrossReboots，activity就有了持久化的能力，
另外需要配合一个新的bundle才行，那就是PersistableBundle。


也就是说，下面的两个方法中的Bundle对象的保存并不是持久化的：

```
protected void onSaveInstanceState(Bundle outState)
protected void onRestoreInstanceState(Bundle savedInstanceState)
```


持久化保存数据，使用下面的方法，注意观察它们的参数。

```
@Override
public void onCreate(@Nullable Bundle savedInstanceState, @Nullable PersistableBundle persistentState) {
  super.onCreate(savedInstanceState, persistentState);
}
@Override
public void onRestoreInstanceState(Bundle savedInstanceState, PersistableBundle persistentState) {
  Debug.i("持久化恢复数据");
}
@Override
public void onSaveInstanceState(Bundle outState, PersistableBundle outPersistentState) {
  Debug.i("持久化保存数据");
}
```

不过默认情况下这三个方法都不会执行, 需要给需要执行的Activity增加一个属性android:persistableMode="persistAcrossReboots"



## Activity保存数据

[Activity的数据保存与恢复 | 设计师吴彦祖](http://liangjingkanji.coding.me/2017/01/15/ActivityStateSave/ "Activity的数据保存与恢复 | 设计师吴彦祖")


默认情况下，系统会使用Bundle对象去保存activity布局中的每个view对象的信息（如刚输入在activity中的文本值）。所以当activity被销毁和重建，布局的状态会自动恢复到之前的状态。
但是如果你的activity需要恢复更多的信息，比如成员变量信息，则需要自己动手写了。





## Fragment

两个Fragment之间的通信，设置目标Fragment，调用目标Fragment的onActivityResult()方法。
没错Fragment中也有onActivityResult(),见[文档](https://developer.android.com/reference/android/app/Fragment.html#onActivityResult)

 startActivityForResult(Intent, int)

见项目中与日期选择相关的实现。


setArguments(Bundle args)


### Fragment数据保存和恢复

保存使用：
```
onSaveInstanceState(Bundle outState)
```

恢复数据重写以下方法：
```
void onCreate (Bundle savedInstanceState)
void onActivityCreated (Bundle savedInstanceState)
```


> 在横竖屏切换或者Activity重建时, Activity其依附的Fragment也会同样的销毁重建.
一般会在AndroidManifest的Activity标签中加入属性`android:configChanges="orientation"`禁止重新创建.

该方法在《Android编程权威指南》中在讲解如何保存WebView数据时的相关章节有介绍，
意思应该是自己实现配置变更为水平时自己处理（此时Activity不会重启）

> 该属性表示，设置不触发生命周期 `android:configChanges="keyboardHidden|screenSize|orientation"`
> 属性值依次代表 键盘隐藏|屏幕尺寸|屏幕方向. 设置以后这三种变化都不会触发生命周期.

### Fragment对象保存和恢复

**Fragment对象**保存和恢复

#### Fragment的非中断保存setRetainInstance：

主要用来保存大量数据。

如果想叫自己的Fragment即使在其Activity重做时也不进行销毁那么就要设置setRetainInstance(true)。
进行了这样的操作后，一旦发生Activity重组现象，Fragment会跳过onDestroy直接进行onDetach（界面消失、对象还在），
而Framgnet重组时候也会跳过onCreate，而onAttach和onActivityCreated还是会被调用。需要注意的是，
要使用这种操作的Fragment不能加入backstack后退栈中。并且，被保存的Fragment实例不会保持太久，
若长时间没有容器承载它，也会被系统回收掉的。

void setRetainInstance (boolean retain)

boolean getRetainInstance();

[Fragment的非中断保存setRetaineInstance | 小蒋的博客](http://www.jiangwenrou.com/fragment%E7%9A%84%E9%9D%9E%E4%B8%AD%E6%96%AD%E4%BF%9D%E5%AD%98setretaineinstance.html "Fragment的非中断保存setRetaineInstance | 小蒋的博客")



> 另见《Android权威编程指南》第319页，三种不同的生命周期的对比： Activity对象、被保留的fragment、Activity的记录（包括使用onSaveInstanceState(Bundle)方法保存的记录）



### Activity与Fragment之间的实时通信

之前了解的都是在Activity或Fragment在创建或销毁时传递数据的方法，现在要考虑在创建之后如何进行通信。

一般的方法：
```
在activity中获取fragment中的数据

1.通过FragmentManager获取fragment布局
FragmantA fragmantA=(FragmantA) manager.findFragmentById(R.id.fragment1);
2.获取在fragment上的视图
View fragmanView=fragmantA.getView();
3.获取视图上的控件
TextViewtextView=(TextView) fragmanView.findViewById(R.id.frag_tv);
4.更改控件上的数据
textView.setText("hhehehehehheh");
================================================================
在fragment中获取activity中的数据

1. 在Fragment中可以通过getActivity的方法来获取activity中的布局
2.调用Activity中的findViewByBd来获取布局中的控件
TextViewtextView=(TextView) getActivity().findViewById(R.id.main_tv);
3. 改变数据
textView.setText("呵呵呵呵呵呵呵");
```

利用回调接口：



> 过渡动画、添加到返回栈

隐藏和显示Fragment：

- FragmentTransaction hide (Fragment fragment)
- FragmentTransaction show (Fragment fragment)

替换容器：

容器即Avtivity上的布局控件, 必须是ViewGroup的子类。容器只是作为占位符，提供一个空间供Fragment显示视图，
但是Fragment无法覆盖里面现有的View；所以容器中不应有其他View。

Fragment无法替换（覆盖）Activity中的视图。


## Dialog对话框

AlertDialog.Builder. 属于构造器模式用法

Builder是属于AlertDialog的内部类. 负责创建AlertDiglog的构造器. 所以属于链式编程.
正因为是构造器模式, AlertDialog的所有方法你都可以直接忽略, Builder已经实现了所有的功能. 并且AlertDialog是Protected权限无法直接创建对象的.


简单的使用：
```
AlertDialog.Builder builder = new AlertDialog.Builder(MainActivity.this);
builder.setTitle("标题");
builder.setMessage("信息主体");
builder.show();
```

详情见印象笔记中的 Android对话框详解

3种用于对话框的按钮：positive按钮(肯定)、negative按钮(否定)、neutral按钮(中立)



### ProgressBar
ProgressBar是作为控件使用的进度条. 且只能写在布局文件中使用. ProgressDialog的实现原理就是创建一个包含了ProgressDialog的View显示.

而INVISIBLE和GONE的主要区别是：当控件visibility属性为INVISIBLE时，界面保留了view控件所占有的空间；而控件属性为GONE时，界面则不保留view控件所占有的空间。

```
//隐藏并不保留progressBar占用的空间
mProgressBar.setVisibility(View.GONE);
```

## Retrofit


自定义Converter来实现更复杂的需求，只需要extends Converter.Factory然后重写


想要使用OkHttp为Retrofit提供更高的定制性，给Retrofit设置自定义的OkHttpClient就可以了

[Retrofit使用指南](http://www.jianshu.com/p/91ac13ed076d "Retrofit使用指南 - 简书")


### (cache)缓存机制

无网络时，也能显示数据

- [Android Retrofit 2.0 使用-补充篇](http://wuxiaolong.me/2016/06/18/retrofits/ "Android Retrofit 2.0 使用-补充篇 | 吴小龙同學")
- [Retrofit2.0+okhttp3缓存机制以及遇到的问题 - Picasso_L的专栏 - 博客频道 - CSDN.NET](http://blog.csdn.net/picasso_l/article/details/50579884 "Retrofit2.0+okhttp3缓存机制以及遇到的问题 - Picasso_L的专栏 - 博客频道 - CSDN.NET")
- [使用Retrofit和Okhttp实现网络缓存。无网读缓存，有网根据过期时间重新请求 - 简书](http://www.jianshu.com/p/9c3b4ea108a7 "使用Retrofit和Okhttp实现网络缓存。无网读缓存，有网根据过期时间重新请求 - 简书")
- [Retrofit2.0+Okhttp不依赖服务端的数据缓存 | 好文](http://dandanlove.com/2016/09/18/retrofit-okhttp-cache-offline/ "Retrofit2.0+Okhttp不依赖服务端的数据缓存 | 下雨天要逛街")



基本原理：

1. 配置Okhttp的Cache
2. 配置请求头中的cache-control或者统一处理所有请求的请求头
3. 云端配合设置响应头或者自己写拦截器修改响应头中cache-control

1. 我们所用的接口服务不支持缓存，所以我不能只修改头信息而让服务端返回的response响应体去实现数据本地缓存。当然在没有网络的情况下我们可以尝试去读取缓存。
2. 因为服务端没有提供response响应体的缓存，所以我们清除response响应体的Pragma、Cache-Control信息，然后根据自己设定的request请求体中的Cache信息去修改response响应体的Cache信息从而达到数据可以缓存。
3. 在开发的过程中遇到如果一个接口在某次请求返回404，那么以后的结果总是请求失败的404页面。所以在请求失败的时候需要初始化OkHttpClient实例。


实现步骤：

1. 开启OKHttp缓存
2. 设置拦截器（缓存）拦截Request
3. 设置Response



## CollapsingToolbarLayout

```
    //使用CollapsingToolbarLayout必须把title设置到CollapsingToolbarLayout上，设置到Toolbar上则不会显示
        CollapsingToolbarLayout mCollapsingToolbarLayout = (CollapsingToolbarLayout) findViewById(R.id.collapsing_toolbar_layout);
        mCollapsingToolbarLayout.setTitle("test");
        //通过CollapsingToolbarLayout修改字体颜色
        mCollapsingToolbarLayout.setExpandedTitleColor(Color.WHITE);//设置还没收缩时状态下字体颜色
        mCollapsingToolbarLayout.setCollapsedTitleTextColor(Color.GREEN);

```

相关配置的作用见：

[Material Design之CollapsingToolbarLayout使用 - 【博客地址永久迁移到】：http://zhengxiaoyong.me - 博客频道 - CSDN.NET](http://blog.csdn.net/u010687392/article/details/46906657 "Material Design之CollapsingToolbarLayout使用 - 【博客地址永久迁移到】：http://zhengxiaoyong.me - 博客频道 - CSDN.NET")



## WebView
高性能 webkit 内核浏览器。

两篇不错的文章，另还可参考《Android编程权威指南》

[深入讲解WebView——上|开源实验室-张涛](https://www.kymjs.com/code/2015/05/03/01/ "深入讲解WebView——上|开源实验室-张涛")

[Android开发：最全面、最易懂的Webview使用详解 - 简书](http://www.jianshu.com/p/3c94ae673e2a# "Android开发：最全面、最易懂的Webview使用详解 - 简书")



### WebView保存状态

You can save the Webview in Android Cache : Use WebView Setting
```
//表示优先使用缓存
webView.getSettings().setCacheMode(WebSettings.LOAD_CACHE_ELSE_NETWORK);
```


### WebView的缓存

在项目中如果使用到 WebView 控件,当加载 html 页面时,会在/data/data/包名目录下生成 database 与 cache 两个文件夹（我的手机没有root，就不截图了）。
请求的 url 记录是保存在 WebViewCache.db,而 url 的内容是保存在 WebViewCache 文件夹下.


### CSS注入、加载本地CSS

loadDataWithBaseURL(String baseUrl, String data, String mimeType, String encoding, String historyUrl)

该方法是loadData(...)的增强版，它不会产生乱码。

- data: 指定需要加载的HTML代码
- mimeType :指定HTML代码的MIME类型，对于HTML代码可以指定为text/html
- encoding: 指定HTML代码编码所采用的字符集

使用示例：` mWebView.loadDataWithBaseURL(null, sb.toString(), "text/html", "utf-8", null);`


方法一：直接在现有html数据之前添加一段导入css的标签


> 方法一和方法二都参考了：
[android - Rendering（渲染 HTML in a WebView with custom CSS - Stack Overflow](https://stackoverflow.com/questions/4950729/rendering-html-in-a-webview-with-custom-css "android - Rendering HTML in a WebView with custom CSS - Stack Overflow")


```
htmlData = "<link rel=\"stylesheet\" type=\"text/css\" href=\"style.css\" />" + htmlData;
// lets assume we have /assets/style.css file
webView.loadDataWithBaseURL("file:///android_asset/", htmlData, "text/html", "UTF-8", null);
```

And only after that WebView will be able to find and use css-files from the assets directory.

ps： And, yes, if you load your html-file form the assets folder, you don't need to specify a base url.


方法二：使用jsoup库

[jsoup Java HTML Parser, with best of DOM, CSS, and jquery](https://jsoup.org/ "jsoup Java HTML Parser, with best of DOM, CSS, and jquery")

[Cookbook: jsoup Java HTML parser](https://jsoup.org/cookbook/ "Cookbook: jsoup Java HTML parser")

jsoup: Java HTML Parser

```
// jsoup HTML parser library @ https://jsoup.org/
compile 'org.jsoup:jsoup:1.10.3'
```

看起来很简单：一个简单示例，
```
Document doc = Jsoup.connect("http://en.wikipedia.org/").get();
Elements newsHeadlines = doc.select("#mp-itn b a");
```

```
//1. load the web-page with jsoup:
doc = Jsoup.connect("http://....").get();

//2. remove links to external style-sheets
doc.head().getElementsByTag("link").remove();

//3. set link to local stylesheet。假设文件在assets文件夹,文件名为 style.css
// <link rel="stylesheet" type="text/css" href="style.css" />
doc.head().appendElement("link").attr("rel", "stylesheet").attr("type", "text/css").attr("href", "style.css");

//4.make string from jsoup-doc/web-page:
String htmldata = doc.outerHtml();

5. display web-page in webview:
WebView webview = new WebView(this);
setContentView(webview);
webview.loadDataWithBaseURL("file:///android_asset/.", htmlData, "text/html", "UTF-8", null);
```


> 方法三的参考： [inject CSS to a site with webview in android - Stack Overflow](https://stackoverflow.com/questions/30018540/inject-css-to-a-site-with-webview-in-android "inject CSS to a site with webview in android - Stack Overflow")

方法三，使用JS操作DOM：

```
public class MainActivity extends ActionBarActivity {

  WebView webView;

  @Override
  protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);

    webView = new WebView(this);
    setContentView(webView);

    // Enable Javascript
    webView.getSettings().setJavaScriptEnabled(true);

    // Add a WebViewClient
    webView.setWebViewClient(new WebViewClient() {

        @Override
        public void onPageFinished(WebView view, String url) {

            // Inject CSS when page is done loading
            injectCSS();
            super.onPageFinished(view, url);
        }
    });

    // Load a webpage
    webView.loadUrl("https://www.google.com");
}

// Inject CSS method: read style.css from assets folder
// Append stylesheet to document head
private void injectCSS() {
    try {
        InputStream inputStream = getAssets().open("style.css");
        byte[] buffer = new byte[inputStream.available()];
        inputStream.read(buffer);
                inputStream.close();
                String encoded = Base64.encodeToString(buffer, Base64.NO_WRAP);
                webView.loadUrl("javascript:(function() {" +
                        "var parent = document.getElementsByTagName('head').item(0);" +
                        "var style = document.createElement('style');" +
                        "style.type = 'text/css';" +
                        // Tell the browser to BASE64-decode the string into your script !!!
                        "style.innerHTML = window.atob('" + encoded + "');" +
                        "parent.appendChild(style)" +
                        "})()");
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
```

但是这里的代码和方法可能有些问题，Stack Overflow上的讨论。

Use DOM manipulation before loading Url in webview. By doing so you can inject any script or javascript file at runtime and then load the content into webview.I have been doing so in my ePub player and it works like charm


另一个示例：

```
private void carregaCSS() {
        final String extraStyles;
        extraStyles= "javascript: "
        + "function css(){ "
            + " document.querySelectorAll('div.live-search-container')[0].style.display = 'none';"
            + " document.querySelectorAll('div#top-bar')[0].style.display = 'none';"
            + " document.querySelectorAll('span#btn-mobile-toggle')[0].style.display = 'none';"
            + " document.querySelectorAll('div#cart')[0].style.display = 'none';"
            + " document.querySelectorAll('div.col-md-7')[0].style.display = 'none';"
            + " document.querySelectorAll('div.breadcrumb')[0].style.display = 'none';"
        + "}"
        +"css();"

        +"";
        mWebView.loadUrl(extraStyles);
        }
}
```

好吧这里我想问的是可以在同一个WebView中多次使用loadUrl加载多个资源吗？？莫非上面是先加载了html页面，再加载该JS代码？？



### JS调用Android方法：

在WebView的JavaScript中调用Android方法的步骤：

- 调用WebSettings的setJavaScriptEnabled(true)启用JS调用功能
- 调用WebView的addJavascriptInterface(Object object, String name)方法将object对象保留给JS脚本
- 在JS脚本中通过刚才暴露的name对象调用Android方法。

```
//Activity中：

mWebView.getSettings().setJavaScriptEnabled(true);
mWebView.addJavascriptInterface(new MyObject(this), "myObj");


//MyObject是自定义的java类，代码如下：
public class MyObject{
    Context mContext;
    ...

    //该注解表示该方法会暴露给JS脚本调用
    @JavascriptInterface
    public void showToast(String name){
        Toast.makeText(...).show();
    }

    @JavascriptInterface
    public void showList(){
        ...
    }
}
```

HTML页面对应的JS代码：

```
...
<body>
    <input type="button" value"打招呼"
        onclick="myObj.showToast('孙悟空');" />
    <input type="button" value="图书列表"
        onclick="myObj.showList();" />
</body>
...
```


## Sqlite

回顾数据库知识：  [SQLite 教程](http://www.runoob.com/sqlite/sqlite-tutorial.html "SQLite 教程 | 菜鸟教程")




## TabLayout

在代码中设置选中某tab：



```
TabLayout tabLayout = (TabLayout) findViewById(R.id.tabs);
TabLayout.Tab tab = tabLayout.getTabAt(someIndex);
tab.select();
```

```
void selectPage(int pageIndex){
    tabLayout.setScrollPosition(pageIndex,0f,true);
    viewPager.setCurrentItem(pageIndex);
}
```




[使用返回和向上导航 | Android Developers](https://developer.android.com/design/patterns/navigation.html?hl=zh-cn#into-your-app "使用返回和向上导航 | Android Developers")

[可绘制对象资源 | Android Developers](https://developer.android.com/guide/topics/resources/drawable-resource.html#StateList "可绘制对象资源 | Android Developers")

drawable


## 测试 removeAllViews()

Whats difference between removeAllViews() and removeAllViewsInLayout()

[android - Whats difference between removeAllViews() and removeAllViewsInLayout() - Stack Overflow](https://stackoverflow.com/questions/11952598/whats-difference-between-removeallviews-and-removeallviewsinlayout "android - Whats difference between removeAllViews() and removeAllViewsInLayout() - Stack Overflow")


removeAllViews() : -Call this method to remove all child views from the ViewGroup.

removeAllViewsInLayout() : -Called by a ViewGroup subclass to remove child views from itself,
when it must first know its size on screen before it can calculate how many child views it will render.

所以在有些情况下，removeAllViews()能移除掉子视图，但removeAllviewsInLayout()移除不掉，因为子视图还未计算。


Fragment无法替换Activity中的视图。


[重写setContentView实现多个Activity部分UI布局相同 - 泡在网上的日子](http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2014/0331/1608.html "重写setContentView实现多个Activity部分UI布局相同 - 泡在网上的日子")


## Style和Theme

toolbar设置主题：

```xml
    <android.support.v7.widget.Toolbar
        android:id="@+id/collection_toolbar"
        android:layout_width="match_parent"
        android:layout_height="?attr/actionBarSize"/>

        <!-- 由于没有添加 android:background="?attr/colorPrimary" 上面的toolbar背景色为白色。 -->
        <!-- 但是添加了这一行后，字体颜色居然成了黑色 -->
        <!-- 使用 android: 命名空间无效的 就用 app命名空间 -->
```










## TODO
~~TODO CardView的点击效果还没有实现。~~

~~设备配置变更后，回到之前的列表位置~~。暂时使用保留fragment对象的方法解决。

~~层级导航，重建Activity问题~~

~~收藏夹排序问题，数据表中添加自动增长的主键，读取时倒序读取，~~ 并在读取时考虑使用分页读取。

~~豆瓣列表的分页读取，每次读取显示10个项目。~~ 略，因为实现了网络缓存。

~~WebView加载css~~

TabLayout在水平布局时，两个tab紧靠在一起。

设置界面功能：

- 删除缓存功能：删除Retrofit，和WebView的缓存


夜间模式
toolbar主题设置

加入高德地图的定位功能。[示例中心 | 高德开放平台](http://lbs.amap.com/dev/demo "示例中心 | 高德开放平台")