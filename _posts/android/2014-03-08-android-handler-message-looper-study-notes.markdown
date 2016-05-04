---
layout: post
title:  "Android的消息机制与线程"
date:   2014-03-08 11:00:03
categories: android
supertype: career
type: android
---

这个是android的线程通信的消息机制，很多，比较复杂，handler，handlerThread，AsyncQueryHandler，message，messageQueue，looper。  
我不是研究系统级别的，就是理解一下，知道怎么使用，大概就差不多了。

这是比较原生的用法了，异步的话，AsyncTask其实就是封装了handler之类的（Looper，HandlerThread）。而handler之类的其实也是封装了java的线程或者C的线程吧。

android的主线程就是UI线程，只能在UI线程对UI进行操作，当其他的工作线程对UI进行操作时候，会报错，所以android使用消息机制解决。给UI线程发一个消息，UI线程取出消息，然后进行UI操作。有消息就有消息队列和消息的发送和处理。

Message就是一个消息的封装，MessageQueue就消息队列，Handler就是发送消息和处理消息的对象。而Looper是管理消息队列的对象，附属于UI线程，附属的意思就是，UI线程有自己的Looper对象，其实是启动线程的时候，自己创建了Looper的，并放到ThreadLocal来保证本地安全的，这个ThreadLocal以后再学习。其实每个线程都可以有Looper，HandlerThread就是一个例子。

我从网上找了下面一张图，其实这图不完整。左边的Thread就是我们的工作线程，它持有UI线程的handler对象，handler对象持有UI线程的Looper对象，工作线程用handler来post一个runnable对象或者一个消息的时候，就是加入到MessageQueue消息队列，Looper就是循环消息队列，取出消息交给对应的handler来处理，执行handleMessage方法，或者执行runnable对象。这样更新UI就不会报错，因为Looper是属于UI线程内部的。一般来讲handler也属于UI线程内部的，当然也可以不是，只要在创建handler对象时候传入UI线程的Looper就可以。  
![handler][handler]

**HandlerThread**  
这个是继承Thread的，和普通线程不同的是，有Looper对象和不会销毁；有点像UI线程，但是又不能去修改UI。因为这个线程的Looper对象不销毁，所以这个线程也不会销毁，所以要退出线程的方法就是调用Looper的quit方法即可。使用它我们就可以使用Handler了，不用创建很多线程。不能修改UI，但是可以做耗时操作，然后发message给UI线程的handler来修改UI。

**Looper**  
Looper.prepare()方法是初始化。  
Looper.looper()是启动工作，不断地读取队列。
Looper.myLooper()获得当前线程的looper。
Looper.getMainLooper()获得主线程的looper对象，就是UI线程。

**MessageQueue**  
队列，里面放Message对象，和Runnable对象。handler往队列放东西，当从队列取出东西以后，也要交给放进去的那个handler来处理。
Looper处理消息不能并发，所以handler的工作不能阻塞。

> PS:注意一下，使用Handler其实是顺序队列处理消息，所以有顺序的事情不要用到java的线程池去做，那里并发不保证顺序的，例如把一个cancel和start操作放到线程池，那么就会出现先cancel再start的情况了。

**Handler**  
构造Handler就需要指定一个Looper，不然不知道消息发到哪里去。默认的构造方法获取的Looper是当前线程的Looper，就是Looper.myLooper()。
当然也可以给Handler指定UI线程的Looper了。这样就不用再UI线程里面来new Handler了。

使用方法就不详细说了，就两种，用handler发Runnable或者Message。不重写的话，一般在UI线程里面写Runnable对象，然后用handler来post，
这样也能修改UI。重写Handler的话就可以发送Message了，需要重写handlerMessage()方法。

公司里面重写了一个抽象类Handler，继承Handler，使用注解的方式，这样就可以new这个自己的handler，然后不用重写handleMessage了，使用注解就OK。具体怎么做的还没研究，因为注解的实现我现在有点忘记了。

由于AsyncTask是封装handler来实现的，而且生命周期比较短，绑定到UI线程。公司里面也使用handler和handlerThread来封装异步任务对象来使用，这个比较简单。

Looper取出Message对象，message对象里面有handler的引用和runnable的引用。如果Handler在UI线程创建，并重写了handleMessage方法来修改UI，但是给这个Handler传非UI线程的Looper。那么looper取出消息时候，是回调handler来修改UI，这个不会报错的，因为handler在UI线程创建，属于UI线程。但是Runnable就不行了，因为looper是直接调用这个runnable的run方法的，runnable是属于Looper所在的非UI线程，即使这个runnable在UI线程创建，它也不属于UI线程的（和直接在UI线程创建一个Runnable来修改UI一个意思！）。

**其他**  
Handler.removeCallbacksAndMessages(null);  
这个方法是移除所有的callbacks 和 message。

[handler]: /image/handler.png