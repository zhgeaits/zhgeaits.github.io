---
layout: post
title:  "Android的应用性能优化，方法数优化，内存分析学习与记录!"
date:   2015-03-17 10:06:03
categories: android
type: android
---

### 性能优化 ###

- **代码优化**

> 减少创建对象的数量。  
优化算法，少使用递归，使用迭代。  
使用缓存效率更高。Android定义了SparseArray类，当key是整数时，它比hashmap效率更高，因为hashmap用的是Integer对象，而SparseArray使用的是int基本类型。使用LruCache是不错的选择，缓存也可以先丢弃那些重建开销很小的项目。  
优化的基本原则是保持应用的持续响应，让主线程尽可能快地完成任务。  
Android提供了工具StrictMode来检测不良行为。开发时候用这个很不错的哦。

- **图形**
  
> 优化布局原则：减少创建对象数量，消除不必要的对象，或推迟创建对象。  
使用<merge/>标签和合并布局。因为Activity的顶是FrameLayout。  

----------

### 方法数、属性数优化 ###
  
> 虽然android程序用java来写，但是android跑的不是java虚拟机，java代码编译成的class文件不用直接用，还需要编译成为dex文件，如下图所示。  
网上很多相关的资料介绍，主要是因为android对于方法和field都是使用short类型来建立索引，所以只能2^16是最大值，很容易就超了。

- **相关命令(在ShellUtility项目下还有详细的shell脚本)**
  
  
**把jar包转换成dex文件的命令如下**    
> dx --dex --verbose --no-strict --output=temp.dex D:\dfm-0.3.1\classes.jar  

**统计dex文件里面的方法数，属性数**  
  
> dexdump.exe -f D:\android-sdk-windows\build-tools\21.1.2\temp.dex | find "method_ids_size"  
> dexdump.exe -f D:\android-sdk-windows\build-tools\21.1.2\temp.dex | find "field_ids_size"  

dx和dexdump命令都在android-sdk-windows\build-tools\21.1.2目录下。

- **属性数例子**

公司的项目方法数达到了5W，属性数达到了3W，但是用idea编译的时候，却奇怪了，方法数增加，属性数也增加，而且属性数增加得离谱，多了几万，一下子就超了65535，想了很久都不知道为什么，只知道是idea的bug，我也还是菜鸟一个，然后公司的一个高手就分析dex文件，对比idea编出来的包和maven命令编出来的包，用dexdump命令来分析数据，排序，过滤，对比，发现多处的是R文件来的属性，只要发现子项目没有r文件，就从application项目复制一个r文件来编译，这样导致属性数就多了。所以大家都想到，搞一个r文件给子项目就好了，然后就随便搞一个string.xml，就会生成r文件了，那样就O了。主要是学习怎么分析dex文件，这个比较重要。。

![alt build_dex](/image/build_dex.png "build_dex") 

----------

### 内存优化 ###

- **使用eclipse的DDMS对android程序进行实时的内存检测**
  
打开eclipse的DDMS视图，在Devices视图选择想要看的进程，然后点击Update Heap，在点击Cause GC，点击Cause GC按钮，相当于向虚拟机请求了一次gc操作，当内存使用信息第一次显示以后，无须再不断的点击Cause GC。然后看heap就可以，观测内存的变化，Heap视图界面会定时刷新，在对应用的不断的操作过程中就可以看到内存使用的变化。

Heap视图中Type列有一行叫做data object，也就是我们的程序中大量存在的类类型的对象，Total Size列的值就是当前进程中所有Java数据对象的内存总量，一般情况下，这个值的大小决定了是否会有内存泄漏。我们可以这样操作：  
1. 不断的操作当前应用，同时注意观察data object的Total Size值。  
2. 正常情况下Total Size值都会稳定在一个有限的范围内，也就是说由于程序中的的代码良好，没有造成对象不被垃圾回收的情况，所以说虽然我们不断的操作会不断的生成很多对象，而在虚拟机不断的进行GC的过程中，这些对象都被回收了，内存占用量会会落到一个稳定的水平。    
3. 反之如果代码中存在没有释放对象引用的情况，则data object的Total Size值在每次GC后不会有明显的回落，随着操作次数的增多Total Size的值会越来越大，直到到达一个上限后导致进程被kill掉。  

- **使用MAT（MemoryAnalyzerTool）进行内存分析**  
  
**MAT安装：**  
可以选择eclipse插件的方式安装  
http://download.eclipse.org/mat/1.3/update-site/  
也可以选择单独MAT程序下载安装  
http://www.eclipse.org/mat/downloads.php

**Eclipse中使用**
  
在上的DDMS视图中检测到内存的数据以后，点击Dump HPROF file即可打开MAT了。

**idea或者android studio中dump出内存.hprof文件**

点击左下角的Android  
选中你的程序的包名  
点击 initiates garbage collection on selected vm  
点击 dump java heap for selected client  

最后可以用eclipse直接file打开hprof文件，也可以直接打开mat来打开hprof文件。

**分析方法**
  
点击OQL图标，在窗口输入：
  
> select * from instanceof android.app.Activity
> 
> 并按Ctrl + F5或者!按钮，看看多少activity是泄露没有被回收的。点击一个activity对象，右键选中Path to GC roots，再选中exclude weak/soft references，就可以看到是哪些引用导致activity没有被回收。

切换到Histogram视图，会列出每個class产生了多少個实例，以及占有多大内存，所占百分比。  
可以很容易找出站内存最多的几个类，根据Retained Heap排序，找出前几个。  
可以分不同的维度来查看类的Histogram视图，Group by class、Group by superclass、Group by class  loader、Group by package  
只要有溢出，时间久了，溢出类的实例数量或者其占有的内存会越来越多，排名也就越来越前，通过多次对比不同时间点下的Histogram图对比就能很容易把溢出类找出来。  

切换到Dominator Tree（支配树）视图，会列出每个对象(Object instance)与其引用关系的树状结构，还包含了占有多大内存，所占百分比。
可以很容易的找出占用内存最多的几个对象，根据Percentage（百分比）来排序。  
可以分不同维度来查看对象的Dominator Tree视图，Group by class、Group by class  loader、Group by package  
和Histogram类似，时间久了，通过多次对比也可以把溢出对象找出来，Dominator Tree和Histogram的区别是站的角度不一样，Histogram是站在类的角度上去看，Dominator Tree是站的对象实例的角度上看，Dominator Tree可以更方便的看出其引用关系。

这里可以使用Merge Shortest Paths to GC Roots来看到引用链，和Path to GC roots一样。

PS：  
list objects -- with outgoing references : 查看这个对象持有的外部对象引用。  
list objects -- with incoming references : 查看这个对象被哪些外部对象引用。  
show objects by class  --  with outgoing references ：查看这个对象类型持有的外部对象引用  
show objects by class  --  with incoming references ：查看这个对象类型被哪些外部对象引用   

**例子与经验**  
android不能频繁开Thread来操作，开大量Thread会导致wait()的崩溃和占用大量系统资源导致创建新Thread时系统报因为线程太多无法创建的崩溃。项目小还好，如果项目大了的话，就要好好管理线程了，用线程池比较好，能够重用，不需要重复打开，我们也要避免使用不会退出的死循环线程。另外使用ZGTask来创建一个HandlerThread管理线程比较好，不会重复创建线程，能重用，但是却采用了Looper，这个是一个队列，并发性没那么好。

尽量不要使用Timer，它会创建新的线程，官方建议android内定时任务使用Handler，或者CountDownTimer，这个是用handler实现的。

避免静态对象引用了Activity、Fragment、View，这样会导致无法回收。平时对于view这些，考虑用弱引用。

内部类会对外部类有一个隐式的引用，不然为什么能直接访问外部类的属性？例如，Activity里面的创建一个Handler内部类，这样只要handler一直在消息队列没被销毁，Activity就会泄露了。可以修改Handler为静态的内部类，它不会对外部类有引用。但是，如果这时候handler持有外部类的属性，例如Textview，而这个textview实际上是对activity有引用的，一样会导致activity泄露，所以在handler里面还是需要用WeakReference对Textview包起来。另外可以使用开源的weekhanlder解决。其实最简单的方法是在Activity销毁的时候调用Handler的removeCallbacksAndMessages(null)就行了。这就是为什么idea会提示内部handler的一个错误：this handler class should be static or leaks might occur

 
**打开strict mode**  
这个也可以检查一些内存泄露，例如实例数量超过2个，详细使用StrictModeUtils这个类。


**jstat跟踪**  