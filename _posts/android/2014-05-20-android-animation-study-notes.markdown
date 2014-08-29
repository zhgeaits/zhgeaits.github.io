---
layout: post
title:  "Android的动画Animation学习笔记!"
date:   2014-05-20 11:00:03
categories: android
type: android
---

android的动画主要分3种：  
补间动画：名字难以理解。它作用于view，定义一系列关于位置，大小，旋转和透明度的改变。有缺陷，如果view
的形状改变，那么控件（如按钮）的按事件触发位置还在老位置。  
帧动画：就是一帧帧的播放图片。  
属性动画 ：可以用于任何地方，且没有补间动画的缺陷。