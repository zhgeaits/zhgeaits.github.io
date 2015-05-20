---
layout: post
title:  "Android的Fragment片段，响应式用户界面学习笔记!"
date:   2014-05-28 11:00:03
categories: android
type: android
---

响应式设计总的思想就是通过重新排列页面结构动态改变可见组件的顺序，这通常以为着需要减少可见栏的数量。  
响应式设计并不是仅仅调整大型组件容器的位置，还会调整这些容器内部元素的位置。

说白了就是UI根据不同尺寸的显示来呈现最佳效果，手机和平板不应该分开两个应用（很多都是手机版和HD版），应该是同一个应用。使用到的技术就是可重用组件Fragment。

**Fragment**  
官网教程地址：http://developer.android.com/training/basics/fragments/index.html  
fragment是对UI的封装，然后重用，和activity很相似，继承Fragment，但是不是系统组件，不用在manifest注册，而且不用setContentView方法。
具体生命周期可以看书和官网，一般就是onCreate(),onCreateView()。在onCreate里面初始化，在onCreateView里面加载布局（和activity一样的布局，没什么区别）创建界面。
每个Activity都有一个FragmentManager管理它所包含的Fragment。  
公司里一般使用都是在Activity的布局里面只用一个FrameLayout，然后在onCreate里面替换成Fragment：
{% highlight java %}
fm = getSupportFragmentManager();
ft = fm.beginTransaction();
ft.replace(R.id.login_fragment_container, new LoginFragment());
ft.commit();
fm.executePendingTransactions();
{% endhighlight %}
FragmentTransaction有add，remove，replace方法来操作fragment。因为是UI的一系列操作，所以要事务。  
add的时候可以加一个tag参数，下次就用fm.findFragmentByTag方法来重用，不用再创建了。  
引用网上：  
调用commit()并不立即执行事务.恰恰相反，它将事务安排排期，一旦准备好,就在activity的UI线程上运行。如果有必要，无论如何，你可以从你的UI线程调用executePendingTransactions()来立即执行由commit()提交的事务. 但这么做通常不必要,除非事务是其他线程中的job的一个从属.  
警告:你只能在activity保存它的状态(当用户离开activity)之前使用commit()提交事务.  
如果你试图在那个点之后提交，会抛出一个异常.这是因为如果activity需要被恢复,提交之后的状态可能会丢失.对于你觉得可以丢失提交的状况，使用 commitAllowingStateLoss().    
如果commitAllowingStateLoss返回小于0，再去执行executePendingTransactions。
还可以把fragment加入到activity的返回栈里面，调用addToBackStack就可以。

另外一种使用方式，就是直接在activity的布局文件使用<fragment>标签，不需要上面的事务了，activity启动时就加载这个fragment了。

一般响应式界面的做法是，根据不同尺寸的屏幕设计不同的布局文件（名字相同），例如手机的是一栏界面，平板是二栏，或者三栏都可以。
分别放在不同尺寸的资源文件下。在activity里面加载布局文件时候就会根据设备动态加载。然后代码里面根据findViewById返回是否为null来判断加载的是那个布局文件，设置相应事件和逻辑。

* **Tips**  
考虑到UI被系统回收后，系统又会自动回复情况（调试模式选择不保留活动），一般来讲，Fragment要保留一个空的默认构造函数。如果要传参数的话，应该用Bundle，然后Fragment.setArguments()方法。如果只有有参数的构造函数则会导致程序崩溃。

**getView()**  
不要随便重写这个方法，他会先与onCreateView()方法调用的，搞不好，整个fragment都没有view。

**onAttach**  
fragment有这个方法，是当fragment绑定到他的activity时候回调的。以前遇到一个问题就是当fragment和activity被回收以后，再由系统恢复，这两者之间就失去关联，fragment拿不到他的activity的引用，现在才发现可以从这里去拿。在这里比较安全，以前是在oncreate那里调getActivity(),但是可能为空，不安全。

**fragment回收慢**  
公司的项目，当前activity里面有fragment，当退出activity以后，fragment还没被销毁，导致core回调两次。。。原因不知道，因为只在小米系统出现，解决方法是core回调的时候要判断当前activity是否top的activity。

现在是简单的学习了一下Fragment，它还有很多东西可以学习。以后慢慢再搞。

**public void setUserVisibleHint(boolean isVisibleToUser)**  
当页面对用户可见的时候会调用这里，isVisibleToUser=true，通常在这里可以做一些请求数据的优化等等。

**Troubleshotting**  

**一个崩溃**

{% highlight java %}
java.lang.RuntimeException: Unable to resume activity {XX.XXActivity}: java.lang.IllegalStateException: Recursive entry to executePendingTransactions
	at android.app.ActivityThread.performResumeActivity(ActivityThread.java:2124)
	at android.app.ActivityThread.handleResumeActivity(ActivityThread.java:2139)
	at android.app.ActivityThread.handleLaunchActivity(ActivityThread.java:1672)
	at android.app.ActivityThread.access$1500(ActivityThread.java:117)
	at android.app.ActivityThread$H.handleMessage(ActivityThread.java:935)
	at android.os.Handler.dispatchMessage(Handler.java:99)
	at android.os.Looper.loop(Looper.java:130)
	at android.app.ActivityThread.main(ActivityThread.java:3691)
	at java.lang.reflect.Method.invokeNative(Native Method)
	at java.lang.reflect.Method.invoke(Method.java:507)
	at com.android.internal.os.ZygoteInit$MethodAndArgsCaller.run(ZygoteInit.java:907)
	at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:665)
	at dalvik.system.NativeStart.main(Native Method)
Caused by: java.lang.IllegalStateException: Recursive entry to executePendingTransactions
	at android.support.v4.app.FragmentManagerImpl.execPendingActions(FragmentManager.java:1450)
	at android.support.v4.app.FragmentActivity.onResume(FragmentActivity.java:445)
	at android.app.Instrumentation.callActivityOnResume(Instrumentation.java:1150)
	at android.app.Activity.performResume(Activity.java:3858)
	at android.app.ActivityThread.performResumeActivity(ActivityThread.java:2114)
	... 12 more
{% endhighlight %}

它只在2.3系统上面崩溃，就是一个activity套了一个fragment，activity的布局是一个framelayout，然后这个layout直接被替换成了fragment。百思不得其解，然后我的同事解决了，他把布局修改了一下，activity的布局是一个RelativeLayout，然后包一个LinearLayout，这个线性布局再被替换成fragment。不知道为什么，我网上也查不到，也没去研究android源码，特别是2.3的，问同事怎么解决的，他说他以前好像也遇到过，也看了别的地方都是这样用就不报错，这也是解决问题的一个思路啊。我们猜测，可能是下拉刷新的控件被包了一个StatusLayout，这个layout会被替换成别的fragment所造成的。

**一个OOM引起的RuntimeException**

{% highlight java %}
java.lang.RuntimeException: Unable to pause activity {com.duowan.mobile/com.yy.mobile.ui.channel.ChannelActivity}: android.view.InflateException: Binary XML file line #7: Error inflating class <unknown>
	at android.app.ActivityThread.performPauseActivity(ActivityThread.java:3226)
	at android.app.ActivityThread.performPauseActivity(ActivityThread.java:3185)
	at android.app.ActivityThread.handlePauseActivity(ActivityThread.java:3160)
	at android.app.ActivityThread.access$1000(ActivityThread.java:147)
	at android.app.ActivityThread$H.handleMessage(ActivityThread.java:1299)
	at android.os.Handler.dispatchMessage(Handler.java:102)
	at android.os.Looper.loop(Looper.java:135)
	at android.app.ActivityThread.main(ActivityThread.java:5233)
	at java.lang.reflect.Method.invoke(Native Method)
	at java.lang.reflect.Method.invoke(Method.java:372)
	at com.android.internal.os.ZygoteInit$MethodAndArgsCaller.run(ZygoteInit.java:904)
	at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:699)
Caused by: android.view.InflateException: Binary XML file line #7: Error inflating class <unknown>
	at android.view.LayoutInflater.createView(LayoutInflater.java:637)
	at android.view.LayoutInflater.createViewFromTag(LayoutInflater.java:747)
	at android.view.LayoutInflater.rInflate(LayoutInflater.java:810)
	at android.view.LayoutInflater.rInflate(LayoutInflater.java:813)
	at android.view.LayoutInflater.rInflate(LayoutInflater.java:813)
	at android.view.LayoutInflater.inflate(LayoutInflater.java:508)
	at android.view.LayoutInflater.inflate(LayoutInflater.java:418)
	at com.yy.mobile.ui.channel.WorksFragment.onCreateView(WorksFragment.java:114)
	at android.support.v4.app.Fragment.performCreateView(Fragment.java:1500)
	at android.support.v4.app.FragmentManagerImpl.moveToState(FragmentManager.java:938)
	at android.support.v4.app.FragmentManagerImpl.moveToState(FragmentManager.java:1115)
	at android.support.v4.app.FragmentManagerImpl.moveToState(FragmentManager.java:1097)
	at android.support.v4.app.FragmentManagerImpl.dispatchPause(FragmentManager.java:1909)
	at android.support.v4.app.Fragment.performPause(Fragment.java:1658)
	at android.support.v4.app.FragmentManagerImpl.moveToState(FragmentManager.java:984)
	at android.support.v4.app.FragmentManagerImpl.moveToState(FragmentManager.java:1115)
	at android.support.v4.app.FragmentManagerImpl.moveToState(FragmentManager.java:1097)
	at android.support.v4.app.FragmentManagerImpl.dispatchPause(FragmentManager.java:1909)
	at android.support.v4.app.FragmentActivity.onPause(FragmentActivity.java:412)
	at com.yy.mobile.ui.BaseActivity.onPause(BaseActivity.java:134)
	at com.yy.mobile.ui.channel.ChannelActivity.onPause(ChannelActivity.java:241)
	at android.app.Activity.performPause(Activity.java:6052)
	at android.app.Instrumentation.callActivityOnPause(Instrumentation.java:1294)
	at android.app.ActivityThread.performPauseActivity(ActivityThread.java:3212)
	... 11 more
Caused by: java.lang.reflect.InvocationTargetException
	at java.lang.reflect.Constructor.newInstance(Native Method)
	at java.lang.reflect.Constructor.newInstance(Constructor.java:288)
	at android.view.LayoutInflater.createView(LayoutInflater.java:611)
	... 34 more
Caused by: java.lang.OutOfMemoryError: Failed to allocate a 528 byte allocation with 16777216 free bytes and 80MB until OOM; failed due to fragmentation (required continguous free 65536 bytes for a new buffer where largest contiguous free 61440 bytes)
	at java.lang.reflect.Constructor.newInstance(Native Method)
	at java.lang.reflect.Constructor.newInstance(Constructor.java:288)
	at android.view.LayoutInflater.createView(LayoutInflater.java:611)
	at com.android.internal.policy.impl.PhoneLayoutInflater.onCreateView(PhoneLayoutInflater.java:55)
	at android.view.LayoutInflater.onCreateView(LayoutInflater.java:686)
	at android.view.LayoutInflater.createViewFromTag(LayoutInflater.java:745)
	at android.view.LayoutInflater.rInflate(LayoutInflater.java:810)
	at android.view.LayoutInflater.inflate(LayoutInflater.java:483)
	at com.handmark.pulltorefresh.library.internal.LoadingLayout.<init>(LoadingLayout.java:87)
	... 37 more
{% endhighlight %}

看到这个OOM，google了一下，人家说是系统级别的错误，看这里https://www.reddit.com/r/Android/comments/34549b/poor_ram_management_affecting_the_galaxy_s6_and/。完全不知道怎么解决，而且重现几率低，因为本来这个错误就是内存不足了的，肯定是app出现了内存泄露，所以要去dump内存分析才行。这里看crash log的栈，activity onPause的时候会执行Fragment.performCreateView，这时候不需要再创建一次view了。目前临时解决方法是这个。