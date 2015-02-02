---
layout: post
title:  "Android的Service组件学习笔记!"
date:   2014-03-25 11:00:03
categories: android
type: android
---

服务很容易想明白，但是真正了解又没有多少。只是知道它是后台运行，可以做一下耗时操作，但是为什么当时tongli那个写的一个服务又有问题？应该是自己还是不够懂Android的Service！

tongli那个的功能是进入视频列表后把手机拍摄的视频复制到指定目录。

回头看tongli那个代码，真可笑，自己在MainActivity启动service，然后destory的时候关闭service，然后搞得服务还没执行完就结束了，没注意是自己结束的，就在怪为什么service不能再后台执行？程序一退出就结束了！傻逼的原因是当时临时学习service怎么使用，代码是copy杨天美那本国产android书。。。。。

后来改进的方法是，service的oncreate方法里面注册一个广播接收者，destory时候就解注册；然后启动MainActivity时候就启动服务，destory时候就关闭服务。在广播接收者里面有一个循环的线程，这个线程不停地从一个队列里面取AsyncTask出来执行，广播接收者接收到一个广播以后就new一个AsyncTask放到队列里面，当service停止时候，就停止这个线程。说白了，还是没有改进，在后台异步进行的只是AsyncTask而已。

而且，很不合理，服务的start完全没用，启动程序时候就执行service的oncreate方法，结束就stop服务，还不如在MainActivity里面注册广播接收者。还是跟原来一样，真正在运行的是AsyncTask。而且队列也是并发的，AsyncTask会出现互斥的同步问题。还有就是不能用广播接收者，因为他的生命周期很短，虽然这里交给了AsyncTask去执行，没有阻塞接收者，但是万一资源不足情况，这些AsyncTask和队列的那个线程就会被回收，很不安全的。详细看广播接收者的blog。

当时真是小菜，没明白这些组件，才会写成这样的代码。

为什么不直接用一个java的线程来做？肯定没有用Android的组件高大上！一旦成行退出，这个线程就会变成空的子线程，也不安全。多个线程在复制，怎么解决同步问题。要自己做队列，比较麻烦，不划算。

为什么不直接用AsyncTask？也不好，它一般跟UI挂钩，适用于生命周期短的时候。代茂大量使用这个，我觉得不好。而且还是要解决同步问题。

可以使用Handler+HandlerThread，普通Thread也可以，但是要使用Looper，因为HandlerThread里面已经有Looper了就方便。这里借助了Looper，因为Looper就是管理消息队列的，而且这个队列不是并发的。这里注意Handler和HandlerThread要用单例模式，特别是HandlerThread，不要多次new。看ZGBox里面ZGTask。依然有问题，一旦程序退出以后，又会变成空子线程，如果放到static那里呢？

直接用服务？不要在MainActivity那里关闭服务就可以？然后让服务stopSelf？当时做tongli那个的时候，就不明白为什么service不是后台运行的，因为service不是异步的！！！这点我一直理解错误，以为后台服务就是后台异步执行的，服务最重要的一点是长生命周期！！而且不会因为阻塞UI线程而被关闭，这点和广播接收者不一样，它最长5-10秒。也就是说，在UI线程startService，那么service的onStartCommand里面执行的东西就要在UI线程执行完毕，这样就会卡在那里了。。。

真正解决方法是，也是一般的Service方法，就是在onStartCommand里面启动一个后台线程去进行耗时的操作。service+AsyncTask，因为service是长生命周期，不像activity，所以AsyncTask就可以执行耗时操作了，这也是代茂的一贯做法，但是我不同意，毕竟AsyncTask是设计来异步更新UI的。而且在这个tongli问题上还要解决同步问题。

最好的解决方法是service+单例的Handler+HandlerThread，这样就不会变成空线程了，又解决同步问题。

什么时候关闭这个service？AsyncTask是执行完就会被回收的，HandlerThread我目前还不是十分了解，这个线程是一直常驻，要退出就调用looper.quit()。而service可以一直不关闭，或者所有业务完成后stopSelf。关于这个关闭以后还需要学习的。

现在已经比较了解什么是service了，但是service还有很多要学习的。先记录简单的。

Service是系统级的组件，所以不需要new，在Manifest注册就可以了，可以有Intent-Fliter，启动服务的时候是在UI线程的，因为startService方法是属于Context的。

**onStartCommand**  

**绑定服务**

**前台Service**

**IntentService**  
这个是逐个处理intent的，也可以解决tongli那个问题。