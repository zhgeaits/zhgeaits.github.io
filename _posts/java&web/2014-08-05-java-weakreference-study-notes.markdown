---
layout: post
title:  "Java 弱引用WeakReference， 软引用SoftReference，强引用Strong Reference, 学习笔记!"
date:   2014-08-05 11:00:03
categories: java
type: java&web
---

强引用Strong Reference：平时我们使用的都是强引用。一旦引用数为0，就会被GC回收。

弱引用WeakReference：弱引用就是不保证不被垃圾回收器回收的对象，它拥有比较短暂的生命周期，在垃圾回收器扫描它所管辖的内存区域过程中，一旦发现了只具有弱引用的对象，就会回收它的内存，不过一般情况下，垃圾回收器的线程优先级很低，也就不会很快发现那些只有弱引用的对象。  
使用：WeakReference<T> wf= new WeakReference<T>();

对于WeakReference，也就是说如果你想写一个Java程序，观察某对象什么时候会被垃圾收集的执行绪清除，你必须要用一个reference记住此对象，以便随时观察，但是却因此造成此对象的reference数目一直无法为零，使得对象无法被清除。如果你希望能随时取得某对象的信息，但又不想影响此对象的垃圾收集，那么你应该用WeakReference来记住此对象，而不是用一般的强引用Strong reference。

软引用SoftReference：如果一个对象具有软引用，内存空间足够，垃圾回收器就不会回收它；如果内存空间不足了，就会回收这些对象的内存。软引用可用来实现内存敏感的高速缓存。使用软引用能防止内存泄露，增强程序的健壮性。SoftReference的特点是它的一个实例保存对一个Java对象的软引用，该软引用的存在不妨碍垃圾收集线程对该Java对象的回收。也就是说，一旦SoftReference保存了对一个Java对象的软引用后，在垃圾线程对这个Java对象回收前，SoftReference类所提供的get()方法返回Java对象的强引用。