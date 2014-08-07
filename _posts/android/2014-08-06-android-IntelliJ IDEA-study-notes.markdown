---
layout: post
title:  "Android的IntelliJ IDEA学习笔记!"
date:   2014-08-05 11:00:03
categories: android
type: android
---

ImageView的scaletype属性：这个属性可以修改图片的显示形式来填充画面，默认是CENTER


可回收的图片控件：
RecycleImageView继承自ImageView：可Recycle Bitmap减少OOM的ImageView
这个view存放的是RecycleBitmapDrawable继承自BitmapDrawable.
给RecycleImageView放置Drawable的时候，首先调用getDrawable()方法来获取上一个在这个view的drawable，然后再去设置新的drawable，最后去通知RecycleBitmapDrawable这个drawable的显示状态，它会根据状态来回收。注意到的是，调用getDrawable()方法获取的可能是RecyleBitmapDrawable，也可能是LayerDrawable。如果是layerDrawable的话，只需要一个个提取出来再递归处理即可。
RecycleBitmapDrawable里面的属性包括，cache引用计数，显示引用计数，是否已经显示过。
进行显示和cache的时候计数都要更新，并且检查状态，一旦各个计数归0，并且已经显示过了，并且这个drawable的bitmap没有被回收，则进行回收这个bitmap。
