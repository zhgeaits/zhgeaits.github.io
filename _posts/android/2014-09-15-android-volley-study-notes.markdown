---
layout: post
title:  "Android的官方的框架Volley，异步网络请求框架AsyncHttpClient，图片加载框架UniversalImageLoader等学习笔记!"
date:   2014-09-15 10:06:03
categories: android
supertype: career
type: android
---

异步网络请求的开源框架，大家用得最火的就是Async Http Client了，可以自行去github上获取代码。它可以处理文本、二进制等各种格式的 web 资源，应该可以用来下载大文件。

异步图片加载的开源框架，Universal Image Loader这个最火，而且一直在更新，自行去github，而且使用很简单。

上面两个功能android的官方出了volley的这个框架，功能都集中了。Volley在性能方面也进行了大幅度的调整，它的设计目标就是非常适合去进行数据量不大，但通信频繁的网络操作，而对于大数据量的网络操作，比如说下载文件等，Volley的表现就会非常糟糕。

**volley的使用**  
从官方下载volley的源码：项目不大，有时间自己去学习。。。   
git clone https://android.googlesource.com/platform/frameworks/volley  
因为这个项目是支持多个构建工具的，最简单就是导入eclipse，然后导出jar包即可。  
如果会maven的话，直接在目录下运行mvn clean javadoc:jar source:jar install -Dmaven.test.skip=true即可。  
还可以用ant，gradle来构建。。  
具体使用在androidbox自己进行了封装。  
网上很多学习这个源码的笔记，比较好的有这个：  
http://www.codekk.com/open-source-project-analysis/detail/Android/grumoon/Volley%20%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90


**AsyncHttpClient的使用**  



**UniversalImageLoader的使用**  
去github下载jar包放到自己的项目，或者maven依赖也可以。github上有非常详细的使用。我自己封装了一下来使用，还是以后学习volley以后自己重新实现。

**volley的源码学习笔记**  
Volley不能大量处理数据量大的请求的原因是，一个请求，它是把网络流一次读完，然后放到内存的byte里面。  
初始化volley就是运行一个请求队列池RequestQueue。  
RequestQueue类，它相当于一个请求队列池。  
里面包含了4请求的队列：  
mWaitingRequests//维护了一个等待请求的集合，如果一个请求正在被处理并且可以被缓存，后续的相同 url 的请求，将进入此等待队列。  
mCurrentRequests//维护了一个正在进行中，尚未完成的请求集合。  
mCacheQueue//缓存请求队列。  
mNetworkQueue//网络请求队列。  
处理请求，要么能够从本地完成（内存，文件），要么需要从网络读取。分别两个实现：  
Cache接口（DiskBasedCache），Network接口（BasicNetwork）。PS：要针对接口编程，而不是针对具体实现编程，这样扩展性高。  
还有处理请求完以后响应分发器ResponseDelivery，包含主线程的handler，还有一个线程执行器Executor，里面纯粹是包装了一下handler，没有管理好线程，估计是为了方便以后的扩展。  
两个分发器分别处理不同的请求，CacheDispatcher和NetworkDispatcher，其中NetworkDispatcher默认有四个，而CacheDispatcher只有一个。这些分发器其实就是线程，不断从队列中取出请求来处理。

当一个请求来的时候，首先加入mCurrentRequests队列，如果这个请求不能缓存的话，直接加入mNetworkQueue队列以后就返回。否则如果mWaitingRequests队列已经包含了，就继续加入到mWaitingRequests队列，其中mWaitingRequests是Map<String, Queue<Request<?>>>类型，value是linkedlist。如果mWaitingRequests没有包含这个请求，就往mWaitingRequests队列插入null值，也加入mCacheQueue队列。这些相同的请求，它没有直接丢弃，而是放到mWaitingRequests缓存起来，暂时不知道有什么作用，估计是以后扩展用吧。mCurrentRequests队列也是。  

CacheDispatcher启动的时候首先初始化DiskBasedCache，cache初始化的时候首先读取缓存目录，把缓存的数据都加载到内存，这个写到文件的内容有一套比较完整的文件序列化协议的。  
CacheDispatcher分发器不断读取mCacheQueue队列的请求，读到一个以后就去cache那里获取结果，如果没有或者结果已经过期了就把请求加入到mNetworkQueue队列就可以等待下一个请求了。命中缓存就交给ResponseDelivery来处理了。

NetworkDispatcher从mNetworkQueue队列获取请求后交给mNetwork来处理网络请求，获取得到网络的结果以后就加入mCache，最后交给ResponseDelivery来处理。