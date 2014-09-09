---
layout: post
title:  "Android的ListView学习笔记!"
date:   2014-03-15 11:00:03
categories: android
type: android
---

listview 每个item的高度：发现如果listview的每一项item没有图片的话，不管怎么设置item的布局高度都是无效的，网上查过以后，发现在item的布局里面设置最小高度就可以了。
{% highlight xml %}
android:minHeight="50dp"
{% endhighlight %}

listview去掉边缘滑动阴影和按下颜色（这些效果耗内存）：
{% highlight xml %}
android:overScrollMode="never"
android: listSelector="@android:color/transparent"
{% endhighlight %}

listview作为聊天界面的时候，要自动滚到底部，可以设置一个定时器，然后执行chatList.setSelection(fragmentAdapter.getCount() - 1);就可以。  

遇到一个奇怪问题，在聊天界面，聊天内容如果用的是listview的话，点击输入框弹出键盘以后，listview会自动滚到底部，但是公司的是用PullToRefreshView，虽然是继承listview的，但是就不会滚动到底部，试过用上面的定时器方法，但是时间不好控制。也试过重写Layout来捕获OnResize方法，但是竟然还是无效。。。后来在stackoverflow上面找到了，加入这个属性就行：  
android:transcriptMode="alwaysScroll"  
listview默认应该是normal，估计PullToRefreshView做了手脚。

给listview设置一个黏住，浮起来不动的标题：StickyListHeaders  
github开源：https://github.com/emilsjolander/StickyListHeaders