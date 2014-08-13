---
layout: post
title:  "Android的ListView学习笔记!"
date:   2014-08-05 11:00:03
categories: android
type: android
---

listview 每个item的高度：发现如果listview的每一项item没有图片的话，不管怎么设置item的布局高度都是无效的，
网上查过以后，发现在item的布局里面设置最小高度就可以了。android:minHeight="50dp"

listview去掉边缘滑动阴影和按下颜色（这些效果耗内存）：
android:overScrollMode="never"
android: listSelector="@android:color/transparent"
