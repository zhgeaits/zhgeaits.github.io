---
layout: post
title:  "Android的播放器实现! 从底层重写一个软解码播放器！"
date:   2014-04-01 11:00:03
categories: android
type: android
---

因为tongli这个大项目，重新写了一个播放器，从底层用ffmpeg来解码重写，并不是使用MediaPlayer这个类。

ffmpeg是一个开源的大项目，可以多平台使用，而且有多个版本，还有一个libav，和ffmpeg一样功能，是从ffmpeg脱离出来的。


编译ffmpeg:

第一次编译的时候，网上搜了很多教程，几乎都不行，后来发现国外roman10的就很好使：  
http://www.roman10.net/how-to-build-ffmpeg-with-ndk-r9/

其实编译很简单，一个脚本即可，在我github项目那里有一个脚本。建议在linux下搞。

硬件解码libstagefright：  
ffmpeg的这个还是建议不要搞了，我弄了很久才编译出来，发现只支持android2.3版本，而且ffmpeg自己也不再维护这个模块了，里面太多bug了，不过可以去学习里面怎么实现omx硬件解码的。他专注于搞软解的。libstagefright.so是omx接口实现的，在android系统里面就有个这个库，以后我再去学习omx这个硬件接口了。。。编译的话，很简单，主要跑tools/build_libstagefright这个脚本即可，重要是下载android源码的一些重要头文件和lib，这个翻墙就好下载了。

vlc：  
第一次发现vlc其实是很大的一个项目，连编译都那么麻烦，而且那时候找不到他解码那一段代码就直接放弃不搞了。最近才再搞起来的。因为它支持硬件解码，所以想直接修改里面的代码来加入通利的算法，编译好so库以后，直接传surface就可以做播放器了。。。想得挺好，也花了不少时间来看源码，基本大概流程搞懂了，发现，尼玛，其实vlc已经放弃omx了，硬解码那块使用的是mediacodec，在jni层回调java方法。无语了，因为mediacodec实在是太快了，直接渲染就好了，虽然mediacodec的硬解码也是omx，但是人家是gpu直接渲染。。。所以我修改vlc代码来编译，发现不用mediacodec，用omx硬解码，播放1080P视频，根本卡得不行，就算是用ffmpeg软解码也不行，太卡了，因为里面的流程很复杂，做了很多事情，而且渲染那块，用的是android的nativewindow或者openes。。。卡。。。比我原来的播放器都还卡得不行。

好吧，还是放弃了vlc，但是还是可以去学习他的omx是如何编写的。。。这点以后再去学习。。。

说说他的源码结构：  
因为他是模块化的，所以可以加载很多模块，方便开发，Modules就是一个个的核心模块代码，包括了解码等，omx解码就是在里面，而Src目录比较接近JNI层调用的。Input/Decoder.c就是主的解码入口，DecoderThread是主的解码线程。然后跟下去即可。Video_output目录就是输出渲染那块了。Video_output/Video_output.c里面的ThreadDisplayPicture(),vout_PutPicture()都是关键方法。可以去看看。

最后建议看源码，用sourceInsight挺好的，虽然不是完美。。。。

暂时硬件解码是优化不了tongli那个项目了，还是先自己写一个player以后再说，其实还可以从ffmpeg的多线程解码那里，但是上次测试貌似效果不好。