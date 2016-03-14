---
layout: post
title:  "Android的四大组件之Activity界面学习笔记!"
date:   2014-01-25 15:00:01
categories: android
type: android
---

这虽然是四大组件之一，但是他不是真正的界面，我刚开始学android的时候以为这就是界面。其实它主要处理一些逻辑的东西，什么生命周期，绘制界面等。  
一个setContentView()就把界面画出来了。

实际上，View是真正的界面内容，在xml文件中编写；Window是一个显示屏，显示view的；activity是一个控制器，调用window来显示view，setContentView的实质是：  
getWindow().setContentView(LayoutInflater.from(this).inflate(R.layout.main, null))；

## 生命周期

名字叫Activity，活动，感觉很奇怪啊，明明就是一个界面嘛，直接叫window之类的名字更好么？其实它虽然是一个界面组件，但是最重要的是它有生命周期，是活的，而界面是静态的，顶层已经包含了window这些类。因此只有理解这个才能写得好程序。
  
生命周期：
  
![alt lifecycle](/image/activity_lifecycle.gif "lifecycle")  

对于可见和不可见，当在A打开一个dialog，那么A就只会执行onPause，如果打开的是一个activity，那就会执行onStop了。

Activity生命周期中，onStart, onResume, onCreate都不是真正visible的时间点，真正的visible时间点是onWindowFocusChanged()函数被执行时。  

> public void onWindowFocusChanged(boolean hasFocus)

从onWindowFocusChanged被执行起，用户可以与应用进行交互了，而这之前，对用户的操作需要做一点限制。

## 跳转

**关于startActivityForResult**

当一个Activity有很多个入口和出口的时候，就是它可能由不同多个Activity启动，而且它也可以去到很多不同的Activity的时候，Activity就需要处理多种情况来改变状态了。

Android给我们提供了requestCode和resultCode的方式处理，例如，从A去到B，那么启动Activity的时候传入requestCode：

>context.startActivityForResult(intent, requestCode);

然后当B要关闭的时候，把resultCode传回去：

>setResult(resultCode);  
>finish();

最后在A要重写这个方法，来处理requestCode和resultCode：

{% highlight java %}  
@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
    super.onActivityResult(requestCode, resultCode, data);
    if(requestCode == a) {
        if (resultCode == b) {
            //do something
        }
    }
}  
{% endhighlight %}

这个看起来很简单，当我在一个比较复杂的业务中使用起来的时候，还是需要认真理解一下的，不然的话，你会发现，你定义的requestCode和resultCode很乱。

其实，这两个code只是一个消息而已，A进入B，会带着requestCode，B不处理requestCode，因为不会影响B，只有intent里面的数据才会决定B，B才不会给不同的requestCode传不同的resultCode，因而B只管自己结束时有哪些状态罢了，所以resultCode定义在B，并把resultCode传回去，同时原封不动地隐含传requestCode回去；requestCode是定义在A的，A需要处理是B还是D回来的resultCode来改变状态。这就是状态机啊。核心是A的状态改变，依赖于出口动作所带回来的消息而改变状态，即到B还是到D回来的resultCode。

## 传参

## Window

Window是一个抽象类，官网上说他的唯一的实现是PhoneWindow，  
官网介绍window：It provides standard UI policies such as a background, title area, default key processing, etc.  
window都有一个View，称之为DecorView，是主窗口中的顶级view，实际上就是ViewGroup。  
并不是一个app只有一个window，而是当有东西需要显示在界面上，就需要一个window，就是说一个界面，包括内容，逻辑和窗口，三者同时需要。启动activity需要window，启动dialog需要window，
只要启动的控件有getWindow()方法都需要一个window来显示。那么这些window的管理，增删回收等就需要一个Manager了。  
可以修改设置window的参数，也可以设置window里面的view的参数，  没有认真学习过这些类，是完全不能理解这些参数设置的意义的。  
设置全屏：  
getWindow().setFlags(WindowManager.LayoutParams.FLAG_FULLSCREEN, WindowManager.LayoutParams.FLAG_FULLSCREEN);

WindowManager主要用来管理窗口的一些状态、属性、view增加、删除、更新、窗口顺序、消息收集和处理等。在WindowManager中还有一个重要的静态类LayoutParams.通过它可以设置和获得当前窗口的一些属性。

在自定义一个弹窗的时候，继承Dialog，然后获取window，设置他的大小，位置，例如在底部，然后再设置动画，则可以实现底部弹窗的效果。但是popwindow也挺好用的。


## starting window

在做应用的时候，很多初始化的东西都放在了application里面去，例如初始化一个sdk。然后导致启动第一个activity的时候很慢，会卡在一个黑屏或者白屏的地方。这是因为应用还没有在运行，系统会为这个Activity所属的应用创建一个进程，但进程的创建与初始化都需要时间，于是系统就会先创建一个starting window，或者叫preview window。Starting Window就是一个用于在应用程序进程创建并初始化成功前显示的临时窗口，拥有的Window Type是TYPE_APPLICATION_STARTING。

启动慢这个问题无法解决，因为你的进程就是需要启动那么久，我们只能修改这个starting window的主题来改进体验，就是在第一个activity，如splash activity那里给他单独设置一个theme，为他设置android:windowBackground,android:windowIsTranslucent属性，为透明，或者一张图片也好。

