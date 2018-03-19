---
layout: post
title:  "Android的MediaCodec解码器学习笔记!"
date:   2014-04-10 11:00:03
categories: private
type: private
---

官方的硬解码，但是文档和使用非常不全面，而且不好操作帧。还没认真去学习，下面是官方的例子：  
http://bigflake.com/mediacodec/

以上例子其实都是测试用的，用处还不是很大。做tongli那个项目，一直想用硬件解码，于是尝试了很久这个mediacodec，参考上面的例子来做，例如，mediacodec解码以后直接渲染到一个surface，然后用opengl来读取这个surface到一个buffer，最后把这个buffer复制到surfaceview的surface上去，虽然流程上可以，但是问题太多了，根本不知道mediacodec什么时候渲染结束可以读取帧，而且还有很多其他问题。。。

最后我还是放弃了，mediacodec他是硬件解码，然后直接GPU来渲染的，速度超快，但是这写buffer都是目前android不让你修改的。。。用mediacodec来做普通的播放器还行，做特定的还是搞不了。其实mediacodec解码也是用OMX的，最好还是自己去实现OMX吧。

我也想过一个办法，就是重写surfaceview这些东西来在界面层面上修改像素达到tongli的需求，这样底层就直接MediaPlayer接口就可以了。但是发现目前做不了surfaceview不好修改，至少获取不了surface上像素。。。