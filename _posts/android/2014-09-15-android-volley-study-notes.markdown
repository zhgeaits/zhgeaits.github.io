---
layout: post
title:  "Android的官方的框架Volley，异步网络请求框架AsyncHttpClient，图片加载框架UniversalImageLoader等学习笔记!"
date:   2014-09-15 10:06:03
categories: android
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


**AsyncHttpClient的使用**  



**UniversalImageLoader的使用**  
去github下载jar包放到自己的项目，或者maven依赖也可以。github上有非常详细的使用。我自己封装了一下来使用，还是以后学习volley以后自己重新实现。