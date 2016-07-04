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

## 2. 针对Socket层的框架

这里是用socket(tcp/ip)来实现http协议的框架。

### 2.1 [Apache Http Client](https://hc.apache.org/)

这是Android最开始使用的http框架了，用的是apache，而apache的这个框架是很强大的，API很全，比较重量级，直接用于java平台的，Google修改了一下，适用于Android，我们可以使用AndroidHttpClient这个类。但是这个框架太复杂了，而且效率方面不够高，对于Android底层的团队并不能有效的工作，于是，Android已经废弃不用了。

### 2.2 [HttpURLConnection](https://developer.android.com/reference/java/net/HttpURLConnection.html)

这是JDK里面的类，它是比较底层的框架，很多接口都是很原始的，但是它很轻量级，功能也很齐全，包括了连接池，缓存等等，性能方面比Apache client快了很多，并且对于Android的团队是很有效的工作，但是它之前的版本存在一个bug，就是调用close()方法的时候会关闭连接池的所有的连接。

关于两者之间的比较，Android的团队有一篇[blog](http://android-developers.blogspot.hk/2011/09/androids-http-clients.html)记录。

### 2.3 [OkHttp](http://square.github.io/okhttp/)

它是Square下的项目，基本上是综合了上面的优点，支持http2和spdy，开发高效，性能高等等。它还是实现了ApacheClient和HTTPURLConnection的接口，因此，可以直接替换上面两个的底层实现。

## 3. 再封装一层框架

上面的框架除了apache Http client的api比较好用以外，其他的都是还不够高效开发的，至少没有缓存那些功能，于是，在往上封装一层，成为更好用的框架，如ImageLoader的功能。

### 3.1 [AsyncHttpClient](https://github.com/AsyncHttpClient/async-http-client)

这个是Java通用的框架，包括了异步Http请求和WebSocket。里面使用了netty。曾经用得很火的框架，不过在android上出现了更多好用的框架。已经开源，有空的话还是很值得去学习一下的。

### 3.2 [android-async-http](https://github.com/loopj/android-async-http)

这是基于Apache Http Client实现的，显然已经很久了，不建议使用了，就算使用Volley也好。

### 3.3 [Volley](https://android.googlesource.com/platform/frameworks/volley/)

针对http请求和图片加载，Google官方估计也是觉得这片缺乏了一些东西，于是出了volley的这个框架，它比较轻量和高可扩展。Volley的设计目标就是非常适合去进行数据量不大，但通信频繁的网络操作，而对于大数据量的网络操作，就不太适合。针对源码的学习，在[codeKK上面也有了文章。](http://www.codekk.com/open-source-project-analysis/detail/Android/grumoon/Volley%20%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90)

然后其实Volley的设计很好，基于它扩展出来的东西也有很大，可以去Google，Github上搜。

#### 3.3.1 为什么不适合数据量大的操作？

Volley不能大量处理数据量大的请求的原因是，一个请求，它是把网络流一次读完，然后放到内存的byte里面。在BasicNetwork里面看到这个方法：

>byte[] entityToBytes(HttpEntity entity);

可以知道它每个请求都是把HttpEntity里面读取完数据，然后返回byte，如果是大文件则会爆掉内存。改进的方法可以是：

1.把network开发出来，我们可以实现一个network来一部分一部分读取网络流。

2.为request增加接口处理数据量大的请求，设置标记使用这个接口就不要使用entityToBytes了。

#### 3.3.2 小记

Volley的优点在于设计非常优美，都是基于接口来编程的，即根据抽象来设计，那样封装的事情交给了具体实现，与设计无关。这样非常便于扩展，而且这个实现很轻量，并没有复杂，易于理解。

关于重试策略，是在网络请求的时候捕获的异常来调用的。分别有SocketTimeoutException、ConnectTimeoutException和IOException。

有人是建议用OkHttp替换Volley里面的网络实现，性能会好很多。

### 3.4 [Retrofit](https://github.com/square/retrofit)

它是Square下的产品，跟Volley一样里面的设计很值得学习源码，它主要的是把RestAPI转换成为了java对象！我才知道这跟我之前的一个专利想法很相似！所以它的介绍说的是类型安全的Http客户端。另外，底层是实现用的肯定是OkHttp了。

### 3.5 [NoHttp](https://github.com/yanzhenjie/NoHttp)

这是国人写的一个框架，感觉跟Volley差不多，就是完善了很多的功能，因为还没阅读源码，所以不太好评价，不知道设计上如何。使用的话，还是蛮方便的了。性能方面，底下应该采用的HTTPURLConnection实现，目前上层仅支持http1.1，对于2和spdy好像都不支持的。

### 3.6 [xUtils](https://github.com/wyouflf/xUtils3)

这是国人写的库，里面不仅仅包含网络的功能，还有各种各样的功能，是一个大全库，很值得学习。在[codeKK](http://a.codekk.com/detail/Android/Caij/xUtils%20%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90)有源码学习了。汇聚了很多东西，没必要用上，但是真的值得学习！http这块一开始用的是HttpClient，后来替换为了HTTPURLConnection了。

## 4. 加载图片的框架

网络请求一开始最主要用于图片加载，所以最早的框架是ImageLoader。而Volley已经包含了ImageLoader的功能了，其他的网络层框架可以自己封装为ImageLoader的功能，不过要注意很多缓存的问题。

### 4.1 [UniversalImageLoader](https://github.com/nostra13/Android-Universal-Image-Loader)

当我们需要异步加载显示网络图片的时候，这个真的最经典的了，关于源码的学习，在[codeKK](http://a.codekk.com/detail/Android/huxian99/Android%20Universal%20Image%20Loader%20%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90)上面已经有文章了。

### 4.2 [Picasso](http://square.github.io/picasso/)

这是Square下的产品，底层用的是OkHttp。使用非常的优雅，一行代码即可。

### 4.3 [Glide](https://github.com/bumptech/glide)

它是一个组织写的，这个组织就是Google的员工，这个框架用于一些Google的app里面，国内用得比较少。功能类似于picasso，它说专注于滚动时的流畅性。还支持很多的多媒体，gif，简略图等等功能。

### 4.4 [Fresco](https://code.facebook.com/posts/366199913563917/introducing-fresco-a-new-image-library-for-android/)

它是FaceBook开源的项目，针对缓存方面应该做得更好，具体可以阅读他们的文档了。它分配了ashmem的缓存。因为之前读过一篇fackebook优化的blog（[Improving Facebook on Android](https://code.facebook.com/posts/485459238254631/improving-facebook-on-android/)），他们面向非洲低端机做了很多优化，所以感觉这个库也是这样做的，在小内存的手机上会表现得很好。

### 4.5 比较

关于上面四大框架的比较，[Trinea](http://www.trinea.cn/android/android-image-cache-compare/)有一篇blog做了分析，挺好的。我都没有读过源码就不好说话了。