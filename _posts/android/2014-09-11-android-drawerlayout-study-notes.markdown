---
layout: post
title:  "Android的抽屉效果，DrawerLayout，SlidingMenu，SlidingPaneLayout，ActionBarDrawerToggle学习笔记!"
date:   2014-09-11 10:06:03
categories: android
type: android
---

因为android的UI设计里面有一个Up键，这个设计好奇葩，让人不能理解，跟返回键混淆，然而，在UI设计的时候，必须给用户一个明确的导航。为了让用户更加明确这个UP键的导航左右，使用侧边的导航栏是很不错的一个解决方案。

最早的时候使用的是SlidingMenu，在github上有一个开源的项目，而且可以支持比较老的api版本，很不错：  
https://github.com/jfeinstein10/SlidingMenu

然后2013的Google I/O大会上，官方发布了DrawerLayout，直接就支持了这个抽屉的导航功能，但是需要比较高的API支持。并且Google放弃了SlidingDrawer，直接使用DrawerLayout即可。  
官方教程：http://developer.android.com/training/implementing-navigation/nav-drawer.html