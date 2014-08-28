---
layout: post
title:  "Android的Handler, Message, Looper学习笔记!"
date:   2014-08-05 11:00:03
categories: android
type: android
---

这个是android的线程通信的消息机制，很多，比较复杂，handler，handlerThread，AsyncQueryHandler，message，messageQueue，looper。
我不是研究系统级别的，就是理解一下，知道怎么使用，大概就差不多了。

这是比较原生的用法了，异步的话，AsyncTask其实就是封装了handler的。而handler其实也是封装了java的线程或者C的线程吧。

android的主线程就是UI线程，只能在UI线程对UI进行操作，当其他的工作线程对UI进行操作时候，会报错，所以android使用消息机制解决。
给UI线程发一个消息，UI线程取出消息，然后进行UI操作。有消息就有消息队列和消息的发送和处理。

Message就是一个消息的封装，MessageQueue就消息队列，Handler就是发送消息和处理消息的对象。而Looper是管理消息队列的对象，附属于UI线程，其实每个线程都可以有Looper。

