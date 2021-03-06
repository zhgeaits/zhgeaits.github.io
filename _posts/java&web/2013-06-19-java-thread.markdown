---
layout: post
title: "Java系列之线程并发"
date: 2013-06-19 23:12:54
categories: java
supertype: career
type: java&web
---

**1\.**java的锁是对象锁，一般每个对象都有一个锁，并且有一个计数器(获得锁的意思是进入sychronzied区)，当获得锁后，计数器加一，只有第一个获得锁的任务在继续获得锁，才能够让计数器累加。同一个对象锁才能够实现同步。

**2\.**wait()方法，挂起，并且释放对象锁，与sleep和yeild对比。wait，notify等方法属于object类继承而来，所以这些方法只有在同步的对象上调用有效。  

**3\.**sychronzied锁定同步对象的方法，只有任务（线程）获得锁以后才能执行，否则被阻塞。sychronzied(object){}可以用来锁定临界区，注意的是，这个锁与括号里面的对象锁同步。在非静态的方法加上这个关键字即锁定这个对象，不能锁定变量。在静态方法用这个关键字，即是锁定了这个类对象，叫做类锁，不是对象锁了，即：synchronized(Xxx.class){}。任务退出sychronzied的区域后自动释放锁，但是调用wait被阻塞的线程必须要唤醒才能执行，所以要调用notify等方法。  

**4\.**java的线程是抢占式的，这表示调度机制会周期性地中断线程，将上下文切换到另一个线程，从而为每个线程都提供时间片，使得每个线程都会分配到数量合理的时间去驱动它的任务。  

**5\.**进程是资源分配单位，主要是指分配内存地址空间，线程是调度单位，因为每行代码是一个指令，所以CPU在一个进程里面并发调度指令执行，线程共享进程的资源，就是全局变量和堆空间，但是有自己的私有栈。cpu的时间片也算一种资源，前面的资源是指空间。  

**6\.**Thread.yeild()将cpu从一个线程转移到另一个线程。对于任何重要的控制或者调整应用时，都不能依赖yield()。  

**7\.**使用线程池：CachedThreadPool和FixedThreadPool,SingleThreadExecutor，管理线程高效。以前总不明白线程池的效率问题，因为没有深入去了解，去想过java底层的线程实现，我们知道池的技术就是预先在池里面初始化很多线程，这样就去掉了每次创建线程的开销，但是我总是搞不明白哪里有提高，因为调用线程池的时候都是传入一个线程的，表面上没有什么区别。其实在于Runnable和Thread，先要了解java线程的机制，不管是实现接口还是继承，都要调用Thread的start方法，所以线程的真正开始在于Thread，这里才会涉及到系统模式的切换。以前也只是肤浅认识到Runnable和Thread只是接口和继承的区别，其实表面上他们都是普通的java类，不是一个线程，真没多大的效率区别，只有把他们放入线程执行才有体现。但是如果用于线程池的话，就明显了。创建一个线程池的话，就已经创建了很多线程，底层的系统调用都完成了，后面只要把实现了Runnable接口的任务传进去就能执行了。也就是说，线程池已经有线程了，然后接收Runnable接口的任务来放到线程去执行。因此Runnable更加轻量一些，而Thread类更多的是代表了一个线程，比较重一些。所以在线程里面都是一个个的Runnable任务，而不是Thread的子类。再总结一下线程的抛弃策略，加入线程池里面固定了同时执行5个线程，那么再进来一个任务（Runnable）的时候，只能在队列等待，当队列也满的时候，怎么办？默认的抛弃策略是丢掉任务，就是拒绝新加入的任务，当然还有其他的策略，那就要去看具体的线程池源码了。

**8\.**一个线程可以在其他线程之上调用join()方法（意思是一个线程拿到第二线程的引用，然后调用第二个线程的join()方法），其效果是等待一段时间直到第二线程结束才继续执行。如果某个线程在另一个线程t上调用t.join()，此线程被挂起，直到目标线程t结束才恢复。  

**9\.**可以使用lock对象进行锁定，就像c语言一样,结合condition使用。  

**10\.**线程本地存储ThreadLocal，是一种自动化机制，可以为使用相同变量的每个不同的线程都创建不同的存储。  

**11\.**如果希望线程执行产生返回值，则实现Callable接口，这个一个泛型！使用executorservice时使用submit方法，而不是execute方法。  

**12\.**对一个executorservice的shutdown调用可以防止新任务被提交到这个Exector。  

**13\.**所谓后台线程，是指在程序运行的时候在后台提供一种通用服务的线程，并且这种线程并不属于程序中不可或缺的部分。因此，当所有的非后台线程结束时，程序也就终止了，同时会杀死进程中的所有后台线程。反过来说，只要有任何非后台线程还在运行，程序就不会终止。后台线程在不执行finally子句的情况下就会终止run方法。

**14\.** 因为对于整型int的操作，很多都是不安全的，如果并发处理就会出现问题，例如++i和i++都不是原子性的。对于不是线程安全的问题，一般使用synchronized来实现，但是这个会被阻塞，所以效率有点低。JDK就提供了AtomicInteger来实现快速的操作。具体的实现好像是使用JNI来调用，还有它的非阻塞算法实现，我目前不太清楚，可以参考下面两个url：

http://hittyt.iteye.com/blog/1130990  
http://www.ibm.com/developerworks/cn/java/j-jtp11234/


**Other Tips**  

**关于打日志**  
想要打印日志的时候，把调用栈的调用方法，类名，和调用行数都打印出来，那么可以这样干：  
Thread.currentThread().getStackTrace()[4].getLineNumber();  
Thread.currentThread().getStackTrace()[4].getFileName();  
Thread.currentThread().getStackTrace()顾名思义就是获取到当前调用线程的栈，我们可以打断点看看会得到什么样的列表。  
StackTraceElement[] tests = Thread.currentThread().getStackTrace();  
第一个element是：  
dalvik.system.VMStack.getThreadStackTrace(Native Method)  
第二个element是：  
java.lang.Thread.getStackTrace(Thread.java:579)  
第三个开始就是你程序的调用方法了，所以上面的4不是绝对的，是根据你的程序来决定的。上面的是android上的结果，第一个是虚拟机的调用本地方法，第二个是Thread的调用方法。