---
layout: post
title:  "Android处理UnCatchException, 捕获崩溃异常等学习笔记!"
date:   2014-09-18 10:06:03
categories: android
type: android
---

当android程序出现UncheckedException异常的时候，程序是直接崩溃掉了，那么我们如何捕获这些异常信息，其实只要实现一个接口就可以了。

写一个类CrashHandler 实现这个接口UncaughtExceptionHandler，只要实现一个方法：public void uncaughtException(Thread thread, Throwable ex)；

然后注册这个类CrashHandler：Thread.setDefaultUncaughtExceptionHandler(this);

当发生崩溃异常的时候就会调用uncaughtException了，这个时候我们就可以去保存异常信息到文件。

具体参考我github上面的ZGCrashHandler，在AlmightyZGBox-android。