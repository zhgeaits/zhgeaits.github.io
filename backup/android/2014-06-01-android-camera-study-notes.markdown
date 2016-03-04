---
layout: post
title:  "Android的Camera学习笔记!"
date:   2014-06-01 11:00:03
categories: android
type: android
---

之前做tongli项目的时候，需要定制摄像头拍出3D照片，所以使用过Camera类，不过这都是比较简单的使用了。

google的文档和API：  
http://developer.android.com/guide/topics/media/camera.html  
http://developer.android.com/reference/android/hardware/Camera.html

现在在公司做的项目，类似于美拍的功能，也需要调用Camera来使用了。主要是open了camera以后，preview成功了，然后给camera设置一个callback，来让camera填充图片buffer数据，这样一般没问题，但是有些奇葩手机，set了callback以后，特别是打开了系统摄像机或者其他的摄像机以后，set了callback就完全拿不到数据。preview都失败了，更加不会回调onPreviewFrame了。

猜测是这些rom的api没有再open的时候lock成功，buffer是临界区，在这里就阻塞掉了，然后一直卡主。所以在set了callback以后，要调用一下camera.lock()，问题就解决了。

对于如果有使用到MediaRecorder来录音什么的话，一定要调用unlock和reconnect这些api，可以去看看google的文档。

对焦问题：

init初始化加速问题：

