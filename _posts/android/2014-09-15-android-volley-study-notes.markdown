---
layout: post
title:  "Android网络层框架收集"
date:   2014-09-15 10:06:03
categories: android
supertype: career
type: android
---

## 1. 引子

当我们进行网络请求的时候，特别是http的请求，虽然已经有很多开源的库，如ApacheHttpClient可以使用了，也有AndroidHttpClient，Java底层也有HttpURLConnection，但是对于现在开发来说，还是不够便利，例如加载网络图片，下载文件等等，于是，便出现了很多很多的框架便于我们开发。

## 2. [AsyncHttpClient](https://github.com/AsyncHttpClient/async-http-client)

这个是Java通用的框架，包括了异步Http请求和WebSocket。里面使用了netty。曾经用得很火的框架，不过在android上出现了更多好用的框架。已经开源，有空的话还是很值得去学习一下的。

## 3. [UniversalImageLoader](https://github.com/nostra13/Android-Universal-Image-Loader)

当我们需要异步加载显示网络图片的时候，这个真的最经典的了，关于源码的学习，在[codeKK](http://a.codekk.com/detail/Android/huxian99/Android%20Universal%20Image%20Loader%20%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90)上面已经有文章了。

## 4. [Volley](https://android.googlesource.com/platform/frameworks/volley/)

针对http请求和图片加载，Google官方估计也是觉得这片缺乏了一些东西，于是出了volley的这个框架，它比较轻量和高可扩展。Volley的设计目标就是非常适合去进行数据量不大，但通信频繁的网络操作，而对于大数据量的网络操作，就不太适合。针对源码的学习，在[codeKK上面也有了文章。](http://www.codekk.com/open-source-project-analysis/detail/Android/grumoon/Volley%20%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90)

### 4.1 为什么不适合数据量大的操作？

Volley不能大量处理数据量大的请求的原因是，一个请求，它是把网络流一次读完，然后放到内存的byte里面。在BasicNetwork里面看到这个方法：

>byte[] entityToBytes(HttpEntity entity);

可以知道它每个请求都是把HttpEntity里面读取完数据，然后返回byte，如果是大文件则会爆掉内存。改进的方法可以是：

1.把network开发出来，我们可以实现一个network来一部分一部分读取网络流。

2.为request增加接口处理数据量大的请求，设置标记使用这个接口就不要使用entityToBytes了。

### 4.2 小记

Volley的优点在于设计非常优美，都是基于接口来编程的，即根据抽象来设计，那样封装的事情交给了具体实现，与设计无关。这样非常便于扩展，而且这个实现很轻量，并没有复杂，易于理解。

关于重试策略，是在网络请求的时候捕获的异常来调用的。分别有SocketTimeoutException、ConnectTimeoutException和IOException。