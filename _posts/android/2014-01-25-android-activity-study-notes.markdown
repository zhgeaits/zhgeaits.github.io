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