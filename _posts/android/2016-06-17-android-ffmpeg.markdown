---
layout: post
title:  "Build ffmpeg for Android with libmp3lame and libshine"
date:   2016-06-17 11:00:03
categories: android
supertype: career
type: android
---

## 1. What's FFmpeg

最初知道[ffmpeg](http://ffmpeg.org/)是在大四的时候做tongli那个项目，需要重新写一个播放器，然后了解到需要用ffmpeg来解码视频。根据官网的介绍，它是一个多媒体框架，可以解码、编码、转码、Mux、demux、处理流、过滤器和播放几乎所有的媒体文件，并且能在各种平台上运行。根据我的了解，ffmpeg可以用来做播放器，因为它是软件解码，也可以编码，转码，对视频做特效，截取帧，等等功能。

ffmpeg里面提供了四大工具，ffmpeg，ffserver，ffprobe，ffplay，目前我只了解过ffmpeg和ffprobe，并且对里面的代码进行修改使用，后面再写文章进行记录。其他两个有机会再去学习使用了。

ffmpeg包含了七大模块：libavutil, libavcodec, libavformat, libavdevice, libavfilter, libswscale, libswresample，大部分从名字就能看出是什么，而libavformat主要是提供是mux和demux功能；libswscale是转换相关，libswresample是重新采样的了，跟转换相关，libavdevice我并不太了解。大部分我都不是很了解，更多详细可以去官网了解。

ffmpeg是开源的，几乎全世界的人都在维护，它的代码量有80w行了，非常的庞大，但是我们可以只编译出一些需要的库！因为它大了吧，ffmpeg的人出来搞了一个相对小一点的[libav](http://libav.org/)库，几乎和ffmpeg有着同样的功能，具体我也没有去了解过。

## 2. Compile FFmpeg

还记得以前玩Linux的时候，在上面安装软件的过程，除了可以直接apt-get命令安装运行文件以外，还可以是编译源码来安装，可以回顾一下[这篇blog](http://zhgeaits.me/other/2012/03/06/linux-setup-study-notes.html)。其实就是make工具来编译源码，一个例子是我们那年去编译[subversion](http://zhgeaits.me/other/2011/07/20/svn-study-notes.html)。

在这里，不能够直接编译ffmpeg了，那样子编译出来的并不能在android上运行的，毕竟至少cpu不同，指令集就不同，我们需要用ndk-build来编译，关于这点原因，还是回顾一下之前[关于jni的blog](http://zhgeaits.me/android/2016/06/13/android-jni.html)。这个概念叫交叉编译，我们在pc平台编译出可以在android平台运行的可执行代码，更多可以Google。

### 2.1 编写编译脚本

首先，我们去官网下载一份最新的ffmpeg源码，我选择了最新的v3.1.1版本。解压以后新建一个build_android.sh脚本文件，实际上是[shell脚本](http://zhgeaits.me/other/2013/06/10/shell-vpn-study-notes.html)，里面执行了configure和make的一些命令而已。这些脚本我也是以前从网站学习回来的，然后不断修改为自己合适的。

我的环境是Mac，NDK是r11c。

{% highlight shell %}

#!/bin/bash
#这句很重要，不然会报错 unable to create temporary file in
export TMPDIR=/Users/zhgeaits/develop/resources/ffmpeg/ffmpeg-3.1.1

# NDK的路径，根据自己的安装位置进行设置
NDK=/Users/zhgeaits/develop/android-ndk-r11c

# 编译针对的平台，可以根据自己的需求进行设置
# 这里选择最低支持android-14, arm架构，生成的so库是放在
# libs/armeabi文件夹下的，若针对x86架构，要选择arch-x86
PLATFORM=$NDK/platforms/android-14/arch-arm

# 工具链的路径，根据编译的平台不同而不同
# arm-linux-androideabi-4.9与上面设置的PLATFORM对应，4.9为工具的版本号，
# 根据自己安装的NDK版本来确定，一般使用最新的版本
TOOLCHAIN=$NDK/toolchains/arm-linux-androideabi-4.9/prebuilt/darwin-x86_64

#设置cpu
CPU=armv7-a
#编译以后的安装路径
PREFIX=./android/$CPU
#configure的最后额外参数，跟着CPU变化
ADDITIONAL_CONFIGURE_FLAG="--cpu=$CPU"
#优化的参数
OPTIMIZE_CFLAGS="-marm -march=$CPU -Wno-multichar -fno-exceptions"
EXTRA_CFLAGS="-O2 -fpic -I$PLATFORM/usr/include $OPTIMIZE_CFLAGS"
#这里用到的是android的lib
EXTRA_LIBS="-lc -lm -ldl -llog -lgcc -lz"

function build_one
{
./configure \
--prefix=$PREFIX \
--target-os=linux \
--cross-prefix=$TOOLCHAIN/bin/arm-linux-androideabi- \
--arch=arm \
--sysroot=$PLATFORM \
--extra-cflags="$EXTRA_CFLAGS" \
--extra-ldflags="$EXTRA_LDFLAGS" \
--cc=$TOOLCHAIN/bin/arm-linux-androideabi-gcc \
--nm=$TOOLCHAIN/bin/arm-linux-androideabi-nm \
--enable-static \
--disable-shared \
--enable-cross-compile \
--enable-runtime-cpudetect \
--enable-gpl \
--enable-small \
--enable-asm \
--enable-yasm \
--enable-pthreads \
--enable-armv5te \
--enable-armv6 \
--enable-armv6t2 \
--enable-vfp \
--enable-neon \
--enable-vfpv3 \
--enable-hwaccels \
--enable-memalign-hack \
--enable-optimizations \
--disable-thumb \
--disable-bzlib \
--disable-debug \
--disable-doc \
--disable-htmlpages \
--disable-manpages \
--disable-pod2man \
--disable-podpages \
--disable-txtpages \
--disable-ffmpeg \
--disable-ffplay \
--disable-ffprobe \
--disable-ffserver \
\
$ADDITIONAL_CONFIGURE_FLAG

sed -i '' 's/HAVE_LRINT 0/HAVE_LRINT 1/g' config.h
sed -i '' 's/HAVE_LRINTF 0/HAVE_LRINTF 1/g' config.h
sed -i '' 's/HAVE_ROUND 0/HAVE_ROUND 1/g' config.h
sed -i '' 's/HAVE_ROUNDF 0/HAVE_ROUNDF 1/g' config.h
sed -i '' 's/HAVE_TRUNC 0/HAVE_TRUNC 1/g' config.h
sed -i '' 's/HAVE_TRUNCF 0/HAVE_TRUNCF 1/g' config.h
sed -i '' 's/HAVE_CBRT 0/HAVE_CBRT 1/g' config.h
sed -i '' 's/HAVE_RINT 0/HAVE_RINT 1/g' config.h

make clean
make -j4
make install

}

build_one

{% endhighlight %}

可以看到实际上主要是运行`configure`脚本来配置交叉编译的环境和选项，其他的gcc编译参数可以看jni那篇blog和去网上查询。使用sed命令修改config.h头文件是因为在android平台不支持这些。关于make命令的参数`-j4`表示的是用4核并行构建。

运行上面的脚本，一会就会生成了android目录，里面有lib和include目录，include下的是头文件，lib是编译好的静态库，可以看到有很多的如`libavcodec.a`的文件，由之前的那篇[关于jni的blog](http://zhgeaits.me/android/2016/06/13/android-jni.html)可以知道我们是可以直接使用这些静态库的了。如果我们不需要静态库而是需要so库的时候可以在configure那里修改为：

>--enable-shared --disable-static

另外，想要配置更多configure的参数，可以运行`./configure --help`查看文档，再次运行脚本可以看到在lib目录下生产了很多so文件和链接文件，但是那些so文件名字都是跟有版本号的，我们可以修改configure文件下面几个字段来不生产这些版本号：

	SLIBNAME_WITH_MAJOR='$(SLIBPREF)$(FULLNAME)-$(LIBMAJOR)$(SLIBSUF)'  
	LIB_INSTALL_EXTRA_CMD='$$(RANLIB)"$(LIBDIR)/$(LIBNAME)"'  
	SLIB_INSTALL_NAME='$(SLIBNAME_WITH_MAJOR)'  
	SLIB_INSTALL_LINKS='$(SLIBNAME)'

运行脚本就会生产如`libavcodec.so`的文件了，这样做是可以让我们单独使用每个库，但是通常我们不需要这么做。

通常我们不想要那么多so文件，只需要一个`libffmpeg.so`文件，那么可以在`make install`命令只有加上一个命令来链接各个.a文件成为一个so文件，也就是说要编译为：

{% highlight shell %}

$TOOLCHAIN/bin/arm-linux-androideabi-ld \
-rpath-link=$PLATFORM/usr/lib \
-L$PLATFORM/usr/lib \
-L$PREFIX/lib \
-soname libffmpeg.so -shared -nostdlib -Bsymbolic --whole-archive --no-undefined -o \
$PREFIX/libffmpeg.so \
libavcodec/libavcodec.a \
libavdevice/libavdevice.a \
libavfilter/libavfilter.a \
libavformat/libavformat.a \
libavutil/libavutil.a \
libpostproc/libpostproc.a \
libswresample/libswresample.a \
libswscale/libswscale.a \
-lc -lm -lz -ldl -llog --dynamic-linker=/system/bin/linker \
$TOOLCHAIN/lib/gcc/arm-linux-androideabi/4.9/libgcc.a 

{% endhighlight %}

我这里配置的cpu是`armeabi-v7a`，一般android平台是最低支持v5的，但是我发现大部分机型都是v7的了，就算低端机也是v6以上的了，如果要修改为v6，只需要改这些配置即可：

{% highlight shell %}

CPU=armv6
OPTIMIZE_CFLAGS="-DCMP_HAVE_VFP -mfloat-abi=softfp -mfpu=vfp -marm -march=$CPU -Wno-multichar -fno-exceptions"
--disable-neon
--disable-thumb

{% endhighlight %}