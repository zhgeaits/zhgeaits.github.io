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

ffmpeg是开源的，几乎全世界的人都在维护，它的代码量有80w行了，非常的庞大，但是我们可以只编译出一些需要的库！因为它太大了吧，ffmpeg的人出来搞了一个相对小一点的[libav](http://libav.org/)库，几乎和ffmpeg有着同样的功能，具体我也没有去了解过。

## 2. Compile FFmpeg

还记得以前玩Linux的时候，在上面安装软件的过程，除了可以直接apt-get命令安装运行文件以外，还可以是编译源码来安装，可以回顾一下[这篇blog](http://zhgeaits.me/other/2012/03/06/linux-setup-study-notes.html)。其实就是make工具来编译源码，一个例子是我们那年去编译[subversion](http://zhgeaits.me/other/2011/07/20/svn-study-notes.html)。

在这里，不能够直接编译ffmpeg了，那样子编译出来的并不能在android上运行的，毕竟至少cpu不同，指令集就不同，我们需要用ndk-build来编译，关于这点原因，还是回顾一下之前[关于jni的blog](http://zhgeaits.me/android/2016/06/13/android-jni.html)。这个概念叫交叉编译，我们在pc平台编译出可以在android平台运行的可执行代码，更多可以Google。

### 2.1 编写编译脚本

首先，我们去官网下载一份最新的ffmpeg源码，我选择了最新的v3.1.1版本。解压以后新建一个build_android.sh脚本文件，实际上是[shell脚本](http://zhgeaits.me/other/2013/06/10/shell-vpn-study-notes.html)，里面执行了configure和make的一些命令而已。这些脚本我也是以前从网站学习回来的，然后不断修改为自己合适的。

我的环境是Mac，NDK是r11c。

{% highlight shell %}

#!/bin/bash
#这句很重要，不然会报错 unable to create temporary file in
export TMPDIR=/Users/zhgeaits/develop/resources/ffmpeg/ffmpeg-3.1.1/ffmpegtmp

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

可以看到实际上主要是运行`configure`脚本来配置交叉编译的环境和选项，其他的gcc编译参数可以看jni那篇blog和去网上查询。其中里面的使用sed命令修改config.h头文件是因为在android平台不支持这些。关于make命令的参数`-j4`表示的是用4核并行构建。

运行上面的脚本，一会就会生成了android目录，里面有lib和include目录，include下的是头文件，lib是编译好的静态库，可以看到有很多的如`libavcodec.a`的文件，由之前的那篇[关于jni的blog](http://zhgeaits.me/android/2016/06/13/android-jni.html)可以知道我们是可以直接使用这些静态库的了。如果我们不需要静态库而是需要so库的时候可以在configure那里修改为：

>--enable-shared --disable-static

另外，想要配置更多configure的参数，可以运行`./configure --help`查看文档，而每次运行脚本的时候，都会输出当前的配置，可以认真查看并配置。目前还是不太懂这个警告是什么意思：

>WARNING: /Users/zhgeaits/develop/android-ndk-r11c/toolchains/arm-linux-androideabi-4.9/prebuilt/darwin-x86_64/bin/arm-linux-androideabi-pkg-config not found, library detection may fail.

再次运行脚本可以看到在lib目录下生产了很多so文件和链接文件，但是那些so文件名字都是跟有版本号的，我们可以修改configure文件下面几个字段来不生产这些版本号：

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

最后把so和头文件复制到jni目录，就可以在android中使用了。

## 3. What's libmp3lame

当我们想要编码为mp3的时候，就需要用到mp3的编码库了，ffmpeg是一个大框架，里面虽然有很多编解码的库，但是也有很多是没有的，但是因为它是一个框架，所以很容易往里面集成东西。由于业务的需要，我做一个转码为mp3的功能，本来以为ffmpeg直接就支持，没想到是没有的，需要集成这个[libmp3lame](http://lame.sourceforge.net/)库。官网上没有教程编译android平台的lame，单纯用configure和make命令编译的应该不能使用，需要像ffmpeg那样配置为ndk的交叉编译才可以。好在lame的代码不多，可以直接配置为jni项目直接用ndk编译。

### 3.1 编译libmp3lame

创建一个lame目录，里面创建一个jni目录，然后去官网上下载最新版3.99，解压以后把里面的libmp3lame和include目录复制到jni目录下，然后在jni目录下创建Application.mk和Android.mk。

Application.mk

{% highlight shell %}

APP_BUILD_SCRIPT :=$(call my-dir)/Android.mk
APP_PROJECT_PATH :=$(call my-dir)
APP_MODULES:=mp3lame
APP_PLATFORM:=android-14
APP_ABI:=armeabi armeabi-v7a x86 mips

{% endhighlight %}

Android.mk

{% highlight shell %}

LOCAL_PATH := $(call my-dir)

include $(CLEAR_VARS)

LOCAL_MODULE := mp3lame
LOCAL_CFLAGS := -DSTDC_HEADERS

LOCAL_SRC_FILES := ./libmp3lame/bitstream.c ./libmp3lame/encoder.c ./libmp3lame/fft.c ./libmp3lame/gain_analysis.c \
	./libmp3lame/id3tag.c ./libmp3lame/lame.c ./libmp3lame/mpglib_interface.c ./libmp3lame/newmdct.c \
	./libmp3lame/presets.c ./libmp3lame/psymodel.c ./libmp3lame/quantize.c ./libmp3lame/quantize_pvt.c \
	./libmp3lame/reservoir.c ./libmp3lame/set_get.c ./libmp3lame/tables.c ./libmp3lame/takehiro.c \
	./libmp3lame/util.c ./libmp3lame/vbrquantize.c ./libmp3lame/VbrTag.c ./libmp3lame/version.c

LOCAL_C_INCLUDES := $(LOCAL_PATH)/libmp3lame $(LOCAL_PATH)/include

ifeq ($(TARGET_ARCH_ABI), armeabi-v7a)    
# 采用NEON优化技术    
LOCAL_ARM_NEON := true    
LOCAL_CFLAGS += -mfpu=neon -mfpu=vfpv3-d16
endif

ifeq ($(TARGET_ARCH_ABI), armeabi) 
LOCAL_CFLAGS += -marm -mfpu=vfp -mfpu=vfpv3 -DCMP_HAVE_VFP
endif

include $(BUILD_STATIC_LIBRARY)

{% endhighlight %}

然后命令行中切换到jni目录运行`ndk-build`，当然我们是完全不用创建jni目录的，直接解压lame-3.99以后在里面创建Application.mk和Android.mk，不过直接运行ndk-build是会报错的：

	Android NDK: Could not find application project directory !    
	Android NDK: Please define the NDK_PROJECT_PATH variable to point to it.    
	/Users/zhgeaits/develop/android-ndk-r11c/build/core/build-local.mk:151: *** Android NDK: Aborting    .  Stop.

是因为ndk-build脚本检查当前目录不是jni目录就判断不是android项目了，然后报错，我们可以这样运行即可：

>ndk-build NDK_PROJECT_PATH=. NDK_APPLICATION_MK=Application.mk

此时就算成功运行ndk-build，依然会报另外一个错：

	In file included from /Users/zhgeaits/develop/resources/ffmpeg/lame/jni/./libmp3lame/bitstream.c:36:0:
	/Users/zhgeaits/develop/resources/ffmpeg/lame/jni/./libmp3lame/util.h:574:12: error: unknown type name 'ieee754_float32_t'
	     extern ieee754_float32_t fast_log2(ieee754_float32_t x);
	            ^
	/Users/zhgeaits/develop/resources/ffmpeg/lame/jni/./libmp3lame/util.h:574:40: error: unknown type name 'ieee754_float32_t'
	     extern ieee754_float32_t fast_log2(ieee754_float32_t x);

那是因为android里面不知道`ieee754_float32_t`是什么，并没有这样的宏定义，我们只要在`util.h`里面修改为`float`即可。再次运行ndk-build，就成功了，在obj目录下生产了`libmp3lame.a`的静态库了。不管是静态库还是动态库都是已经可以在[jni](http://zhgeaits.me/android/2016/06/13/android-jni.html)里面使用的了。但是要集成到ffmpeg中去，我们还是使用静态库好。

### 3.2 Build ffmpeg with libmp3lame

在ffmpeg的目录下运行`./configure --help | grep libmp3lame`，可以发现有`--enable-libmp3lame`的选项，说明ffmpeg是已经支持lame的了，完全可以集成进去。于是我们修改编译脚本：

{% highlight shell %}

--enable-libmp3lame
--enable-encoder=libmp3lame
ARMEABI=armeabi-v7a
LAMEDIR=/Users/zhgeaits/develop/resources/ffmpeg/lame
EXTRA_LDFLAGS="-L$LAMEDIR/obj/local/$ARMEABI"
EXTRA_CFLAGS="-O2 -fpic -I$PLATFORM/usr/include -I$LAMEDIR/jni/libmp3lame -I$LAMEDIR/jni/include $OPTIMIZE_CFLAGS"

{% endhighlight %}

>注意上面$LAMEDIR/obj/local/$ARMEABI指向的是libmp3lame.a位置。

重新运行脚本会发现报错`ERROR: libmp3lame >= 3.98.3 not found`，我们去查看`config.log`会发现：

>fatal error: lame/lame.h: No such file or directory

于是在include目录建立lame目录，然后把lame.h复制进去即可。最后就可以编译成功了，注意如果是编译为一个so库，别忘了把libmp3lame.a也要链接进去。

### 3.3 JNI中调用lame接口

不管是直接用libmp3lame的库，还是编译进去ffmpeg里面，只要用到jni里面去，我们都可以直接调lame的接口了。首先添加下面的头文件：

{% highlight c %}

lame_global_flags *initializeDefault(JNIEnv *env);

lame_global_flags *initialize(JNIEnv *env, jint inSamplerate, jint outChannel,
        jint outSamplerate, jint outBitrate, jfloat scaleInput, jint mode, jint vbrMode,
        jint quality, jint vbrQuality, jint abrMeanBitrate, jint lowpassFreq, jint highpassFreq,
        jstring id3tagTitle, jstring id3tagArtist, jstring id3tagAlbum,
        jstring id3tagYear, jstring id3tagComment);

jint encode(JNIEnv *env, lame_global_flags *glf, jshortArray buffer_l, jshortArray buffer_r,
        jint samples, jbyteArray mp3buf);

jint encodeBufferInterleaved(JNIEnv *env, lame_global_flags *glf,
        jshortArray pcm, jint samples, jbyteArray mp3buf);

jint flush(JNIEnv *env, lame_global_flags *glf, jbyteArray mp3buf);

void close_vidmate_lame(lame_global_flags *glf);

{% endhighlight %}

然后实现这些接口：

{% highlight c %}

lame_global_flags *glf;

lame_global_flags *initializeDefault(JNIEnv *env) {
    lame_global_flags *glf = lame_init();
    lame_init_params(glf);
    return glf;
}

lame_global_flags *initialize(JNIEnv *env, jint inSamplerate, jint outChannel,
        jint outSamplerate, jint outBitrate, jfloat scaleInput, jint mode, jint vbrMode,
        jint quality, jint vbrQuality, jint abrMeanBitrate, jint lowpassFreq, jint highpassFreq,
        jstring id3tagTitle, jstring id3tagArtist, jstring id3tagAlbum,
        jstring id3tagYear, jstring id3tagComment) {

    lame_global_flags *glf = lame_init();
    lame_set_in_samplerate(glf, inSamplerate);
    lame_set_num_channels(glf, outChannel);
    lame_set_out_samplerate(glf, outSamplerate);
    lame_set_brate(glf, outBitrate);
    lame_set_quality(glf, quality);
    lame_set_scale(glf, scaleInput);
    lame_set_VBR_q(glf, vbrQuality);
    lame_set_VBR_mean_bitrate_kbps(glf, abrMeanBitrate);
    lame_set_lowpassfreq(glf, lowpassFreq);
    lame_set_highpassfreq(glf, highpassFreq);
    switch (mode) {
        case 0:
            lame_set_mode(glf, STEREO);
            break;
        case 1:
            lame_set_mode(glf, JOINT_STEREO);
            break;
        case 3:
            lame_set_mode(glf, MONO);
            break;
        case 4:
            lame_set_mode(glf, NOT_SET);
            break;
        default:
            lame_set_mode(glf, NOT_SET);
            break;
    }
    switch (vbrMode) {
        case 0:
            lame_set_VBR(glf, vbr_off);
            break;
        case 2:
            lame_set_VBR(glf, vbr_rh);
            break;
        case 3:
            lame_set_VBR(glf, vbr_abr);
            break;
        case 4:
            lame_set_VBR(glf, vbr_mtrh);
            break;
        case 6:
            lame_set_VBR(glf, vbr_default);
            break;
        default:
            lame_set_VBR(glf, vbr_off);
            break;
    }
    const jchar *title = NULL;
    const jchar *artist = NULL;
    const jchar *album = NULL;
    const jchar *year = NULL;
    const jchar *comment = NULL;
    if (id3tagTitle) {
        title = (*env)->GetStringChars(env, id3tagTitle, NULL);
    }
    if (id3tagArtist) {
        artist = (*env)->GetStringChars(env, id3tagArtist, NULL);
    }
    if (id3tagAlbum) {
        album = (*env)->GetStringChars(env, id3tagAlbum, NULL);
    }
    if (id3tagYear) {
        year = (*env)->GetStringChars(env, id3tagYear, NULL);
    }
    if (id3tagComment) {
        comment = (*env)->GetStringChars(env, id3tagComment, NULL);
    }

    if (title || artist || album || year || comment) {
        id3tag_init(glf);
        if (title) {
            id3tag_set_title(glf, (const char *) title);
            (*env)->ReleaseStringChars(env, id3tagTitle, title);
        }
        if (artist) {
            id3tag_set_artist(glf, (const char *) artist);
            (*env)->ReleaseStringChars(env, id3tagArtist, artist);
        }
        if (album) {
            id3tag_set_album(glf, (const char *) album);
            (*env)->ReleaseStringChars(env, id3tagAlbum, album);
        }
        if (year) {
            id3tag_set_year(glf, (const char *) year);
            (*env)->ReleaseStringChars(env, id3tagYear, year);
        }
        if (comment) {
            id3tag_set_comment(glf, (const char *) comment);
            (*env)->ReleaseStringChars(env, id3tagComment, comment);
        }
    }
    lame_init_params(glf);
    return glf;
}

jint encode(JNIEnv *env, lame_global_flags *glf, jshortArray buffer_l, jshortArray buffer_r,
        jint samples, jbyteArray mp3buf) {
    jshort *j_buffer_l = (*env)->GetShortArrayElements(env, buffer_l, NULL);
    jshort *j_buffer_r = (*env)->GetShortArrayElements(env, buffer_r, NULL);
    const jsize mp3buf_size = (*env)->GetArrayLength(env, mp3buf);
    jbyte *j_mp3buf = (*env)->GetByteArrayElements(env, mp3buf, NULL);
    int result = lame_encode_buffer(glf, j_buffer_l, j_buffer_r, samples, j_mp3buf, mp3buf_size);
    (*env)->ReleaseShortArrayElements(env, buffer_l, j_buffer_l, 0);
    (*env)->ReleaseShortArrayElements(env, buffer_r, j_buffer_r, 0);
    (*env)->ReleaseByteArrayElements(env, mp3buf, j_mp3buf, 0);
    return result;
}

jint encodeBufferInterleaved(JNIEnv *env, lame_global_flags *glf,
        jshortArray pcm, jint samples, jbyteArray mp3buf) {
    jshort *j_pcm = (*env)->GetShortArrayElements(env, pcm, NULL);
    const jsize mp3buf_size = (*env)->GetArrayLength(env, mp3buf);
    jbyte *j_mp3buf = (*env)->GetByteArrayElements(env, mp3buf, NULL);
    int result = lame_encode_buffer_interleaved(glf, j_pcm, samples, j_mp3buf, mp3buf_size);
    (*env)->ReleaseShortArrayElements(env, pcm, j_pcm, 0);
    (*env)->ReleaseByteArrayElements(env, mp3buf, j_mp3buf, 0);
    return result;
}

jint flush(JNIEnv *env, lame_global_flags *glf, jbyteArray mp3buf) {
    const jsize mp3buf_size = (*env)->GetArrayLength(env, mp3buf);
    jbyte *j_mp3buf = (*env)->GetByteArrayElements(env, mp3buf, NULL);
    int result = lame_encode_flush(glf, j_mp3buf, mp3buf_size);
    (*env)->ReleaseByteArrayElements(env, mp3buf, j_mp3buf, 0);
    return result;
}
void close_vidmate_lame(lame_global_flags *glf) {
    lame_close(glf);
    glf = NULL;
}
{% endhighlight %}

然后剩下的是在java层定义JNI的接口来调用即可，首先是初始化`init`，然后`encode`，最后`flush`和`close`，流程都非常简单，注意到的是`encodeBufferInterleaved()`函数是交叉存储的意思，即不分左右音轨的意思，有时候我们解码拿到的数据就是交叉存储的，不用专门去拆分左右音轨了。另外，编码是不支持多线程的，可能需要自己去修改源码，应该就是全局变量的问题。

## 4. What's libshine

原本使用libmp3lame这个库就已经足够的了，它已经是业界的权威mp3编码库，非常的标准和稳定，但是效率方面太慢了，就算我针对arm架构cpu的各种特性，neon，vfp等等编译加速，在高端手机上还是蛮快的，但是一旦遇到低端手机以后就非常非常的慢！不能满足需求，于是各种网上google和Github，发现了[shine](https://github.com/toots/shine)，根据说明和测试，它是超级快的一个mp3编码库，特别是在没有FPU的cpu上，使用的是Fixed-point算法，具体我也不懂，不过使用以后，性能真的快了很多！

### 4.1 编译libshine

可以看到Github上的源码它直接是支持android的了，clone下来代码，发现里面有一个Makefile.am文件，去掉后缀名，Makefile文件里面已经有android的目标了，我们修改Binary.mk和Library.mk文件，去掉formatbits.c，因为src目录下并没有这个文件的了。然后直接在shine目录下执行make命令，即可完成编译了，可以看到在android目录生成了libshine.so了。当然我们可以修改Library.mk文件，生成静态库libshine.a而不是so库，不管怎么样，都是可以直接在jni使用的了。

另外，我们完全可以像libmp3lame那样，把代码复制出来，建立jni目录，创建Android.mk和Application.mk文件，然后自行编译。

### 4.2 Build ffmpeg with libshine

比较麻烦的就是要把libshine集成到ffmpeg里面去了。我们在ffmpeg目录运行`./configure --help | grep shine`，发现ffmpeg已经和libmp3lame那样支持的了，我们需要模仿libmp3lame那样修改编译脚本：

{% highlight shell %}

--enable-libshine
--enable-encoder=libshine
ARMEABI=armeabi-v7a
SHINEDIR=/Users/zhgeaits/develop/resources/ffmpeg/shine
EXTRA_LDFLAGS="-L$SHINEDIR/obj/local/$ARMEABI"
EXTRA_CFLAGS="-O2 -fpic -I$PLATFORM/usr/include -I$SHINEDIR/jni/shine $OPTIMIZE_CFLAGS"

{% endhighlight %}

注意到上面的脚本是没有包含libmp3lame的，当然我们是完全可以同时包含两个库的。另外`--enable-encoder=libshine`这个一定要有，不然ffmpeg是不会编译进去shine库的。然后开始编译，你会发现报错找不到shine，然后我们去看`config.log`：

	check_pkg_config shine shine/layer3.h shine_encode_buffer 
	false --exists --print-errors shine 
	ERROR: shine not found using pkg-config

难道又是头文件位置不对？我去修改layer3.h的位置，依然报错！我们去查看configure文件，所有地方lame和shine都是一样的配置，唯独这里不同：

>enabled libmp3lame        && require "libmp3lame >= 3.98.3" lame/lame.h lame_set_VBR_quality -lmp3lame

而shine是这样的：

>enabled libshine          && require_pkg_config shine shine/layer3.h shine_encode_buffer

我再去看`libavcodec/Makefile`和`libavcodec/allcodecs.c`，里面两者的配置都是一样的，我依旧不明白为什么上面的配置不同，我目前也不懂pkg-config是个上面东东！这留给以后去学习好了，我就尝试修改为这样：

>enabled libshine          && require "shine" shine/layer3.h shine_encode_buffer -lshine

再次运行脚本，居然成功了！看来并没有多大的区别和关系啊。我猜测原本的配置是用于检查编译有没有pkg-config的，而这个应该是pc平台，在android平台并找不到。

另外，ffmpeg是怎么把libmp3lame和libshine集成过去的？我们看到`libavcodec/libmp3lame.c`和`libavcodec/libshine.c`文件，里面调用了编码的接口，实现了ffmpeg的标准接口，瞬间明白，这就像插件一样把库插进去了！如果我们没有`--enable-encoder=libshine`这样的配置，那么就不会编译libshine.c文件了！

### 4.3 JNI中调用shine接口

不管是直接用libshine的库，还是编译进去ffmpeg里面，只要用到jni里面去，我们都可以直接调shine的接口了。关于shine的api使用，并没有文档，不过由于比较简单，代码也不多，加上如果对音频多媒体知识有点了解，基本是没有困难的。我们可以查看源码里面`main.c`文件简单的使用把wav转换mp3，还可以查看`layer3.h`头文件对api的说明。

{% highlight shell %}

shine_config_t config;
shine_t s;
int written;

void zhangge_init_shine(int channels, int samplerate) {
    shine_set_config_mpeg_defaults(&config.mpeg);
    config.wave.channels = channels;
    config.wave.samplerate = samplerate;
    /* Set to stereo mode if wave data is stereo, mono otherwise. */
    if (config.wave.channels > 1)
        config.mpeg.mode = STEREO;
    else
        config.mpeg.mode = MONO;
    s = shine_initialise(&config);
}
int get_samples_per_pass() {
    return shine_samples_per_pass(s);
}
unsigned char *zhangge_shine_encode_buffer(int16_t **data) {
    return shine_encode_buffer(s, data, &written);
}
unsigned char *zhangge_shine_encode_buffer_interleaved(int16_t *data, int *byteEncode) {
    unsigned char *result = shine_encode_buffer_interleaved(s, data, byteEncode);
    return result;
}
unsigned char *zhangge_shine_flush() {
    return shine_flush(s, &written);
}
void zhangge_shine_close() {
    return shine_close(s);
}
JNIEXPORT void JNICALL Java_org_zhangge_ShineCodec_shineInit
        (JNIEnv *env, jclass cls, jint channels, jint samplerate) {
    zhangge_init_shine(channels, samplerate);
}
JNIEXPORT jint JNICALL Java_org_zhangge_ShineCodec_shineSamplesPerPass
        (JNIEnv *env, jclass cls) {
    int sample_per_pass = get_samples_per_pass();
    return sample_per_pass;
}
JNIEXPORT void JNICALL Java_org_zhangge_ShineCodec_shineFlush
        (JNIEnv *env, jclass cls) {
    zhangge_shine_flush();
}
JNIEXPORT void JNICALL Java_org_zhangge_ShineCodec_shineClose
        (JNIEnv *env, jclass cls) {
    zhangge_shine_close();
}
JNIEXPORT jbyteArray JNICALL Java_org_zhangge_ShineCodec_shineEncodeInterleaved
        (JNIEnv *env, jclass cls, jshortArray data) {
    short *pcm = (*env)->GetShortArrayElements(env, data, NULL);
    unsigned char *mp3buf = zhangge_shine_encode_buffer_interleaved(pcm, &written);
    jbyteArray result = (*env)->NewByteArray(env, written);
    (*env)->SetByteArrayRegion(env, result, 0, written, mp3buf);
    (*env)->ReleaseShortArrayElements(env, data, pcm, 0);
    return result;
}

{% endhighlight %}

上面就是核心的代码了，对于java层的使用就更简单了，但是它不并lame那么简单，可以任意传一个buffer下去编码，这个buffer的大小需要通过调用get_samples_per_pass()来获取，而written参数是编码成功以后的大小。