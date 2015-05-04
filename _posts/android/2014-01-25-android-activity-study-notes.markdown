---
layout: post
title:  "Android的四大组件之Activity界面学习笔记!"
date:   2014-01-25 15:00:01
categories: android
type: android
---

就是一个界面组件，最重要的是它的生命周期，只有理解这个才能写得好程序。看下图  
生命周期：  
![alt lifecycle](/image/activity_lifecycle.gif "lifecycle")  

对于可见和不可见，当在A打开一个dialog，那么A就只会执行onPause，如果打开的是一个activity，那就会执行onStop了。

**public void onWindowFocusChanged(boolean hasFocus)**  
Activity生命周期中，onStart, onResume, onCreate都不是真正visible的时间点，真正的visible时间点是onWindowFocusChanged()函数被执行时。  
从onWindowFocusChanged被执行起，用户可以与应用进行交互了，而这之前，对用户的操作需要做一点限制。