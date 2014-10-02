---
layout: post
title:  "Android的SurfaceView学习笔记!"
date:   2014-03-20 11:00:03
categories: android
type: android
---

因为tongli这个项目做视频播放，所以需要学习到surfaceview，这里记录一下当时的理解和笔记。

做播放器也学习到了不少，一个activity需要一个view来显示，那么要在界面时显示画面，就把这个surfaceview交给界面即可。view是靠canvas来画出来的，canvas把surface画到界面即可，所以surface里面含有canvas对象，然后把这个对象交给surfaceview来把自己画出来。surface本质是一块内存，视频的帧就是渲染到surface这里。SurfaceHolder是一个监听的操作，里面有回调接口，用来监听surface。

一般来说，播放器都是需要传一个surface过去，解码以后渲染到这个surface来进行显示。但是不知道具体系统是什么时候把一个surface画到surfaceview上的，我猜测是surface被lock，然后unlock以后。。。