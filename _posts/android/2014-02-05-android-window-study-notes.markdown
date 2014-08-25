---
layout: post
title:  "Android的View，ViewGroup，Window，WindowManager学习笔记!"
date:   2014-02-05 11:00:03
categories: android
type: android
---

学习android如果纯粹是学会了简单的开发，是做不出来令人惊讶的界面。不理解这些重要的内部结构，就会在写UI的时候蛋疼无比。

**Activity**  
这虽然是四大组件之一，但是他不是真正的界面，我刚开始学android的时候以为这就是界面。其实它主要处理一些逻辑的东西，什么生命周期，绘制界面等。  
一个setContentView()就把界面画出来了。

实际上，View是真正的界面内容，在xml文件中编写；Window是一个显示屏，显示view的；activity是一个控制器，调用window来显示view，setContentView的实质是：  
getWindow().setContentView(LayoutInflater.from(this).inflate(R.layout.main, null))；

**View和ViewGroup**  
引用网上一句话：Android系统中的所有UI类都是建立在View和ViewGroup这两个类的基础上的。所有View的子类成为”Widget”，所有ViewGroup的子类成为”Layout”。  
View和ViewGroup之间采用了组合设计模式，可以使得“部分-整体”同等对待。ViewGroup作为布局容器类的最上层，布局容器里面又可以有View和ViewGroup。

View是最顶层的界面类，ViewGroup是继承View的抽象类，所以ViewGroup是一组view的集合。view又是所有界面的父类引用，view引用可以指向viewgroup，实现了多态，
在adapter那里getView什么的是很好的体现。

**Window和WindowManager**  
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

**dialog的edittext无法弹出键盘问题**  
//只用下面这一行弹出对话框时需要点击输入框才能弹出软键盘
window.clearFlags(WindowManager.LayoutParams.FLAG_ALT_FOCUSABLE_IM);
//加上下面这一行弹出对话框时软键盘随之弹出
window.setSoftInputMode(WindowManager.LayoutParams.SOFT_INPUT_STATE_ALWAYS_VISIBLE);

暂时就只学习到这里，日后继续补充。

**PopWindow**  
日后再学习