---
layout: post
title:  "Learning Android Network Library OkHttp"
date:   2017-02-23 11:00:03
categories: android
supertype: career
type: android
---

现在OkHttp已经发展到了3.6版本了，虽然一出来就已经知道，但是从来没有去学习和使用，只是简单的了解过一下，也曾经在[这篇blog](http://zhgeaits.me/android/2014/09/15/android-http-study-notes.html)里面关于Android网络框架有过一些记录。今天就来学习和使用一下这个框架吧，毕竟，从Android4.4开始，底层的网络已经使用OkHttp和OkIO来代替了HttpURLConnection的默认实现，[提交的历史在这](https://android.googlesource.com/platform/libcore/+/2503556d17b28c7b4e6e514540a77df1627857d0)。

## 1. What's OkHttp

官方[地址在这](http://square.github.io/okhttp/)，还有[GitHub](https://github.com/square/okhttp)。

虽然4.4以后系统使用OkHttp了，但是低端机还有很多，我们还是需要支持的。当年FaceBook的App以前的优化加载图片就是使用了OkHttp，因为OkHttp 支持在糟糕的网络环境下面更快的重试，并且还能利用SPDY协议进行快速的并发网络请求。

在OkHttp的官网上面介绍得比较清楚，反正就是一堆的高性能啊高效啊的好处，然后google一下OkHttp的教程，基本上都是翻译了一下而已。当然，现在已经很多研究源码的资料了，我现在还没有这个研究源码的兴趣呢。

关于网上的教程，比较好的有[IBM的这篇](https://www.ibm.com/developerworks/cn/java/j-lo-okhttp/)，和[稀土挖金的这篇](http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2015/0106/2275.html)。既然网上都有教程，我也不搬砖，这里只是记录一些自己的理解和别人没有的东西。

## 2.Volley扩展

关于Volley里面扩展支持使用OkHttp，在这个[Gist](https://gist.github.com/JakeWharton/5616899)上有大家的交流。

## 3.封装使用

## 4.关于拦截器