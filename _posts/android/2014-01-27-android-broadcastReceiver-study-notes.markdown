---
layout: post
title:  "Android的四大组件之BroadcastReceiver"
date:   2014-01-27 15:00:01
categories: android
supertype: career
type: android
---
这个东西很简单，但是时间一长总是容易忘记，容易混淆！BroadcastReceiver是android四大组件之一，注意，
是广播接收者，我自己总是想成了广播者和接收者！这样一想就会把这个机制给搞混了。这个机制很简单，跟网络上的ARP
物理广播差不多（是不是arp？忘记了！）。广播的是Context，所以调用Context.sendBroadcast(Intent)就可以，就是说，
广播这一块不用去做什么，在activity或者context的子类实现类就能进行广播，重点是实现一个接收者。

实现Receiver很简单，只要extends BroadcastReceiver，然后重写onReceiver()方法即可。在android里面onXXX的方法一般
是事件触发执行的，这里就是说接收到一个广播以后就会执行这个方法。不像网络那样每个网卡都有一个独立的物理地址，
这里不同的接收者可以有相同的接收条件，就是Intent-filter，就是说每个接收者都有一个过滤条件，发送广播就是发送intent，
接收者根据设置的intent-filter来过滤掉intent，然后对通过的intent进行响应。

因为广播接收者是系统级的组件，就像activity一样，所以使用的话需要进行注册。一种方式，就像activity一样，在androidmanifest.xml
里面注册，加一个<Receiver>标签即可，标签里面还要设置intent-filter，这种注册方式使得receiver不用自己来new，并且
在这个应用程序生命周期中都是活动的。另一种注册方式，是在activity里面注册，这个需要自己来new实现的receiver类，
还要new一个intent filter，然后调用Context的registerReceiver(receiver, filter)来注册，一般在onResume()注册，在onPause()
注销，调用unregisterReciver(receiver)。

很重要的一点是：android给一个receiver的执行时间为5-10秒，所以一般来讲receiver不应该执行耗时的事情！！！  
引用网友一段话：不应该在BroadcastReceiver中开启一个新线程完成耗时的操作，因为BroadcastReceiver本身的生命周期很短，
可能出现的情况是子线程还没有结束，BroadcastReceiver就已经退出的情况，而如果BroadcastReceiver所在的进程结束了，
该线程就会被标记为一个空线程，根据Android的内存管理策略，在系统内存紧张的时候，会按照优先级，结束优先级低的线程，
而空线程无异是优先级最低的，这样就可能导致BroadcastReceiver启动的子线程不能执行完成。
耗时操作应该启动服务来完成（service应该在AndroidManifest里面注册，在activity里面bind，但是退出activity就没用了，之前我遇到的问题还没去研究清楚！）

更多的学习，什么sticky intent, lbm，intent filter等，慢慢去看书。