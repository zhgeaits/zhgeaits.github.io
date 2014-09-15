---
layout: post
title:  "Java AtomicInteger原子整数学习笔记!"
date:   2014-08-02 11:00:03
categories: java
type: java
---

因为对于整型int的操作，很多都是不安全的，如果并发处理就会出现问题，例如++i和i++都不是原子性的。对于不是线程安全的问题，一般使用synchronized来实现，但是这个会被阻塞，所以效率有点低。JDK就提供了AtomicInteger来实现快速的操作。具体的实现好像是使用JNI来调用，还有它的非阻塞算法实现，我目前不太清楚，可以参考下面两个url：

http://hittyt.iteye.com/blog/1130990  
http://www.ibm.com/developerworks/cn/java/j-jtp11234/