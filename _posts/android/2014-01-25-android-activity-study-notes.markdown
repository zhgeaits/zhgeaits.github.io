---
layout: post
title:  "Android的四大组件之Activity界面学习笔记!"
date:   2014-01-25 15:00:01
categories: android
type: android
---

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






