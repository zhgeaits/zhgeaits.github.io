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

**onAttach**  
fragment有这个方法，是当fragment绑定到他的activity时候回调的。以前遇到一个问题就是当fragment和activity被回收以后，再由系统恢复，这两者之间就失去关联，fragment拿不到他的activity的引用，现在才发现可以从这里去拿。在这里比较安全，以前是在oncreate那里调getActivity(),但是可能为空，不安全。

*fragment回收慢*  
公司的项目，当前activity里面有fragment，当退出activity以后，fragment还没被销毁，导致core回调两次。。。原因不知道，因为只在小米系统出现，解决方法是core回调的时候要判断当前activity是否top的activity。

现在是简单的学习了一下Fragment，它还有很多东西可以学习。以后慢慢再搞。