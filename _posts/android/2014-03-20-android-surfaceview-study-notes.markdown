---
layout: post
title:  "Android的SurfaceView学习笔记!"
date:   2014-03-20 11:00:03
categories: android
supertype: career
type: android
---

因为tongli这个项目做视频播放，所以需要学习到surfaceview，这里记录一下当时的理解和笔记。

做播放器也学习到了不少，一个activity需要一个view来显示，那么要在界面时显示画面，就把这个surfaceview交给界面即可。view是靠canvas来画出来的，canvas把surface画到界面即可，所以surface里面含有canvas对象，然后把这个对象交给surfaceview来把自己画出来。surface本质是一块内存，视频的帧就是渲染到surface这里。SurfaceHolder是一个监听的操作，里面有回调接口，用来监听surface。

一般来说，播放器都是需要传一个surface过去，解码以后渲染到这个surface来进行显示。但是不知道具体系统是什么时候把一个surface画到surfaceview上的，我猜测是surface被lock，然后unlock以后。。。

用surfaceview做弹幕，那么需要surfaceview背景透明，那么需要在拿到surfaceview时候调用这几个方法：  
gSurfaceView.setZOrderOnTop(true);  
gSurfaceView.getHolder().setFormat(PixelFormat.TRANSLUCENT);//还有这个TRANSPARENT，前者半透明，后者全透明。  
注意的是，这两个一定要尽早调用，不要再surface的oncreat里面调用，否则无效。  
然后在画画的时候调用这个清屏：canvas.drawColor(Color.TRANSPARENT, Mode.CLEAR);