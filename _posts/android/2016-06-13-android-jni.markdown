---
layout: post
title:  "Android JNI Develop Stuff"
date:   2016-06-13 11:00:03
categories: android
supertype: career
type: android
---

>这里必须先说一个故事，在我大三拿到offer以后决定转行Android，然后决定在大四的时候一边游玩一边写一个Android项目，以备毕业入职之需。然而，在我还没入门的时候就拿到工作室的高大上的项目，做一个裸眼3D播放器，哈哈！我还没熟悉Android的时候就涉足了JNI的开发，那个时候主要涉及的都是底层的C和视频编解码相关的知识！就是没有记录多少在Blog上。那时候用的还是Eclipse，现在工作两年了，早已放弃了而用Android Studio了，现在AS对JNI的支持还是不太好，刚好现在的工作再次涉及到JNI了，所以便有了这篇...

## 1. What's JNI

JNI是Java Native Interface的缩写，即java本地接口。想想，java是运行在虚拟机上，jvm是用c实现的，必然jvm是可以调用c/c++的接口的，也是必须的功能，c写的代码几乎都能在系统层运行，当java上无法实现调用一些系统接口的时候，可以用c来实现，然后java能够调用c的代码，那么就可以解决问题了。而实际上，因为java的很多接口就是用c来实现的，例如打开一个文件，jdk里面的实现是用c来实现的。可以看到JNI本质是提供给开发者自己去调用C的一个接口，java和底层代码之间提供给开发者的通信接口，毕竟不是所有东西都能用java来实现！

## 2. JNI开发流程

要实现java方法调用c函数，这已经是jvm帮我们做好的事情了，我们只需要根据约定来定义接口名字即可，剩下的链接问题交给了jvm。

### 2.1 定义Java的Native方法

这个很简单，只需要在方法前面加一个`native`关键字修饰即可，方法内部不用实现，可以是静态也可以是非静态。如下：

>public native void dispHelloWorld();

### 2.2 定义C的头文件

C语言靠的是头文件进行连接的，所以要生成约定形式的头文件，jdk里面已经包含了工具`javah`，切换到src目录以后执行下面的命令就可以了：

>javah -jni org.zhangge.jni.HelloWorld

然后就会生成一个`org.zhangge.jni.HelloWorld.h`头文件了，里面包含了这样的函数声明：

>/*
> * Class:     org_zhangge_jni_HelloWorld_dispHelloWorld  
> * Method:    dispHelloWorld  
> * Signature: (V)V  
> */  
>JNIEXPORT void JNICALL Java_org_zhangge_jni_HelloWorld_dispHelloWorld(JNIEnv *, jobject);

还有详细的注释说明，可以看到函数的一些规律，以Java和包名、类名开头的，还会有方法签名。剩下的我们只需要根据这个头文件进行开发即可。

### 2.3 编译链接

**1. 以下是linux的编译过程：**

>gcc -fPIC -D_REENTRANT -I $JAVA_HOME/include -I $JAVA_HOME/include/linux -c jni_HelloWorldImpl.c

`-D_REENTRANT`是用于多线程的，先不管

`-f`后面跟一些编译选项，PIC是其中一种，表示生成位置无关代码（Position Independent Code）

其他是include的引入

再链接成链接库，命令：

>gcc -shared jni_HelloWorldImpl.o -o libhelloworld.so

`-shared`代表的是生成so动态链接库

>命名：lib是必须的，helloworld是loadLibrary指定的。

最后生成`libhelloworld.so`。

**2. 以下是编译win下的dll动态链接库，需要vs，cl就是vs的命令：**

最后执行编译命令：

>cl -I %JAVA_HOME%/include -I %JAVA_HOME%/include/win32 -LD jni_HelloWorldImp.cpp -Fe hellodll.dll

`-I`表示编译包含的额外目录，`-LD`表示产生dll，`-Fe`后面表示产生dll的名字

如果没有vs就安装mingw

>gcc -Wall -shared mingw_dll.c -o mingw_dll.dll -I %JAVA_HOME%/include -I %JAVA_HOME%/include/win32

最后生成`hellodll.dll`。

### 2.4 加载链接库

只需要在调用java native之前加载库即可，一般我们会在静态代码块里面去加载，如下：

{% highlight java %}

static {
	System.out.println(System.getProperty("java.library.path"));
//		System.loadLibrary("hellodll");
	System.loadLibrary("helloworld");
}

{% endhighlight %}

### 2.5 C还是C++

进行JNI开发的时候，即可以用C语言，也可以用C++语言，因为C++是对C的扩展，一般来说，C++的代码.cpp文件里面是可以直接兼容c的，但是如果要混合的话，是会出现很多问题的，所以，我建议使用一种就好了，我自己是熟悉c的，基本是用C进行开发的。

当我们用javah命令生成的头文件以后，可以发现有这么一段代码：

{% highlight c %}

#ifdef __cplusplus
extern "C" {
#endif

//functions defines here

#ifdef __cplusplus
}
#endif

{% endhighlight %}

这个`extern "C"`包含了我们的函数的原型定义。原本extern关键字是修饰变量的，使得我们可以获取外部的变量，而这里结合了“C”来使用，告诉编译器这些函数按照C的方式来编译，那就不会出现错误了。关于extern这一块的知识，还是蛮多的，因为我不做c++开发，所以没有深入理解。

`__cplusplus`是C++的自定义宏定义，我不知道怎么搞来的，只要后缀是cpp的时候就会有这个宏定义了。我们看到jni.h里面有这样的定义：

{% highlight c %}

#if defined(__cplusplus)
typedef _JNIEnv JNIEnv;
typedef _JavaVM JavaVM;
#else
typedef const struct JNINativeInterface* JNIEnv;
typedef const struct JNIInvokeInterface* JavaVM;
#endif

{% endhighlight %}

可以知道，如果是cpp文件的话，JNIEnv实际上就是c++的对象，如果是c文件的话，就是一个结构体了。如果没有搞懂这些的话，使用JNIEnv的时候就不会注意到使用的是结构体还是对象了，会出现这样的一个报错：

>applying operator -> to JNIEnv instead of pointer

我建议使用c开发就可以了。

### 2.6 捕获异常

在JNI底下也是可以抛出异常的，具体可以看JNIEnv里面定义的方法。我在使用里面的一个FindClass方法的时候，如果没有找到class就会抛出ClassNotFound的异常，我想要捕获，然后清除这些异常信息，可以这样做：

{% highlight c %}

jclass activityThread = (*env)->FindClass(env, "android/app/ActivityThread");
if ((*env)->ExceptionCheck(env)) {
    (*env)->ExceptionClear(env);
    return;
}

{% endhighlight %}

这样以后就不会出现异常信息和crash了，可以让程序继续跑并且可以保护信息不让别人看到。

## 3. Android JNI开发

在Android上也是可以进行jni开发的，因为android也是基于linux系统来的，也是可以在dvm里面跑c的代码，不过需要用到ndk里面的`ndk-build`来编译，并不是说随便拿一个so库来就可以执行的，必须通过ndk编译过的才能在android系统上运行，具体估计跟编译环境和系统等等有关系，我还没学习过相关知识，所以无法解释了。[ndk-build](https://developer.android.com/ndk/guides/ndk-build.html)是Google提供的一个编译脚本。

__为什么要去写native代码？__

首先是需要实现java无法实现的功能，有些时候可能是java没有api支持，那么我们如果对c比较熟悉的话就可以去写c来实现了；其次，当我们需要提高性能的时候，用c写的效率是会比较高的，就是如果要实现性能优化的时候可以考虑jni开发；如果开发的功能依赖到第三方的库来实现，如ffmpeg，shine, sdl等等，这些都是c来实现的，那么我们就需要移植到android，然后自己再用jni进行开发了。再就是想要动态加载库的时候，我们可以把核心模块写到c里面，不需要集成到apk里面去，当运行到这里的时候才让用户进行下载so库加载（jar包可以么？）；还有，如果我们想要保护核心的功能不被破解，是可以把功能写到so里面的，破解so至少比破解jar难一些，不过也是可以的，对于安全这块又是一大块领域了。最后，因为开发的so是native代码实现的，几乎是可以在其他平台上重用的！

关于一些ndk的概念，可以从[官网](https://developer.android.com/ndk/guides/concepts.html#hiw)上阅读更多。而关于NDK更多官方的例子可以看[Github](https://github.com/googlesamples/android-ndk/tree/android-mk)。

### 3.1 环境

对于底层开发，Android是提供了NDK的，里面最主要的是编译工具，直接去官网下载，然后和SDK一样，解压，然后配置NDK_HOME环境变量即可，那样就可以直接在命令行运行相关命令的了。需要注意到的是，不是最新版就好的，很多情况下，最新版采用新的编译环境，导致编译出来的文件会有各种问题，以前做ffmpeg开发的时候就遇到过，因为知识太少，所以完全无从下手，所以一般要配置到比较旧的版本；然而官网只提供了最新的下载链接，所以，呵呵哒，自己去网上找，还要保存起来。

### 3.2 Eclipse配置

官方最初推荐的开发IDE，很多东西支持得都很好，特别是对于JNI的支持，但是UI方面可能没有IDEA好。直接右键项目，选择add native support来添加jni的支持，就可以了，然后就会生成一个jni目录，我们编写的c文件都放在这个目录下，那么每次运行项目的时候就会运行ndk-build的了，编译jni目录下的c代码，并且会把libs目录清空，重新把编译的库install进去！如果有第三方的库，需要配置到jni目录下面去，那样就会每次都install进去了。

#### 3.2.1 配置javah命令

在eclipse里面可以配置javah命令，然后选中java文件运行相关选项即可生成.h头文件了。

>Run->External Tools->External Tools Configurations.

配置如下：

	Name:Generate Header File  
	Location:${system_path:javah}  
	Working Directory: ${project_loc}/jni/player/include  
	Arguments:-classpath "${project_classpath};${env_var:ANDROID_SDK_HOME}/platforms/android-16/android.jar" ${java_type_name}  

然后切换到`Refresh`标签，勾上`Refresh resources upon completion`。再选上`The project containing the seleced resource`。最后切换到`Common`标签，勾上`External Tools`。

实际会比较麻烦，还有各种问题，我个人比较喜欢直接自己写命令，可以写成脚本即可，还可以熟练命令行！

#### 3.2.2 native调试

在Eclipse里面进行native调试，很简单，最新版的adt已经支持了，在build command那里改成`ndk-build NDK_DEBUG=1`，然后右键项目，`Debug as->Android native application`就可以了！可是android4.3有一个bug，一直run-as不了，我没成功，然后用2.3也没试成功，也没有别的手机了，就不试了。有空再去这里研究一下：[ndk-build](https://developer.android.com/ndk/guides/ndk-build.html)，[ndk-gdb](https://developer.android.com/ndk/guides/ndk-gdb.html)。

#### 3.2.3 关于Android.mk和Application.mk文件

在jni目录下写了c文件以后，就算运行ndk-build，它并不会像java那样自动编译所有的c文件的，如果我们学习过c编译会知道make工具，它是GNU的编译工具，需要根据Makefile来进行编译链接代码。看到后缀名mk便知道是make的缩写，android的ndk既然是编译c，也是需要make的，官方文档说这两个是GNU Makefile的片段，暂时还是不太懂这些的了。另外，android的配置文件一般都是有继承关系的，可以去到ndk目录下找到很多父类的.mk文件，而且语法比较完善。

全局jni目录下只有一个`Application.mk`，这个文件列举和描述了我们app所需要的模块，里面主要包括了编译的abi，toolchains和包含的标准库(static、dynamic STLport或者default system)。

我们至少需要一个`Android.mk`文件，当有多个模块的时候就需要多个Android.mk文件了。里面定义了模块和它的名字，需要编译的文件，build flags和需要链接的库，每次运行ndk-build脚本命令的时候必然会去寻找这个文件进行编译的。通常有些变量可以在Application.mk全局定义，会应用到每一个Android.mk。

### 3.3 Android Studio配置

在as2.0以后对jni的支持已经相对够好了，像SDK一样配置NDK的目录即可，结合gradle基本上能完成基本的开发，因为使用了gradle，所以一般来说，是不需要Android.mk和Application.mk配置文件的了。但是对于第三方库的时候，却很多地方无法配置，无奈我只能去掉jni的支持，自己手动执行ndk-build来编译。

#### 3.3.1 添加JNI支持

只需要在main目录下，右键选择new，在`Folder`那里选择`Add Jni Folder`即可，其实直接new一个名字为jni的目录也是可以的。gradle在构建的时候就会去检查并执行ndk命令的了。

然后把javah生成的头文件复制到jni目录，再去编写c文件就可以的了。生成的so库不再是install到外面的libs目录，libs目录一般放的是第三方的jar包了。它会install到了build目录`intermediates/ndk`下的了。

接着在`build.gradle`文件的defaultConfig节点下加入下面配置：

{% highlight gradle %}

ndk{
    moduleName "helloworld"
    abiFilters "armeabi", "armeabi-v7a", "x86", "mips"
    ldLibs "log"
}

{% endhighlight %}

可以看到，其实配置的内容和两个.mk文件是一样的，上面的字段都比较容易理解，点进去还能看到更多的信息。而本质上，gradle还是把这个脚本编译生成了两个mk文件的。

运行的时候应该会报错的，as对jni的支持还不够好，我们在`gradle.properties`文件加上这一行即可：

>android.useDeprecatedNdk=true

最后项目运行成功以后，生成的so库在`build/intermediates/jniLibs`目录下。

#### 3.3.2 Windows版本Android Studio的Bug

在mac上跑as，运行jni项目还是比较正常，没什么大问题的，但是一旦换到windows下就出现了两个问题，一个是项目路径太长，另外一个是不能编译单个文件。

我的jni项目放在D盘的workspace下的某某目录，然而，实际上也不是很深的，不过，本来gradle构建的目录结构也是不浅的。run的时候就报这个错误了：

>Error:(45, 1) opening dependency file D:\workspace\xxx\aaa\bbb\build\intermediates\ndk\release\obj/local/armeabi-v7a/objs/bbb/D_\workspace\xxx\aaa\bbb\src\main\jni\org_zhgeaits_ccc.o.d: No such file or directory

>Error:Execution failed for task ':bbb:compileReleaseNdk'.
>com.android.ide.common.process.ProcessException: Error while executing 'D:\develop\android-sdk-windows\ndk-bundle\ndk-build.cmd' with arguments {NDK_PROJECT_PATH=null APP_BUILD_SCRIPT=D:\workspace\xxx\aaa\bbb\build\intermediates\ndk\release\Android.mk APP_PLATFORM=android-22 NDK_OUT=D:\workspace\xxx\aaa\bbb\build\intermediates\ndk\release\obj NDK_LIBS_OUT=D:\workspace\xxx\aaa\bbb\build\intermediates\ndk\release\lib APP_ABI=armeabi-v7a,armeabi,mips,x86,arm64-v8a}

当时遇到这个问题我也是懵逼了，明明是同样的项目在mac上可以执行，而且查看了as官方所有文档，已经用的是最新版了，也说明了特性，已经比较好的支持JNI开发了。为毛还是报错？我google了很多这个错误，几乎都没法解决，网上的解决方法都是用于下面的另外一个错误。于是，我就新建一个jni项目，居然正常了运行啊？把c代码复制过去，也正常啊，再把demo的项目名称修改成为一样的，也是OK啊。。。这就神奇了。后来突然灵机一动，难道是路径太长了？于是把项目复制到D盘根目录。。。好吧，problem solved！

另外一个问题也是windows版本as的bug，当你新建一个jni项目的时候，如果只有一个.c或者.cpp文件，注意，不包括.h头文件。运行一样报上面的错误，所以我搜上面的error log的时候全部都是叫我新建一个empty.c文件就可以了。然后我测试了这个问题，确实是存在的。

#### 3.3.3 去掉jni的支持

当我要使用第三方so库的时候，可以是复制到libs目录，但这是不合理的用法，另外，如果使用的是.a静态库并要加入到jni的编译中去的时候就没法做了。因为很多Android.mk的配置，在gradle上还没有支持，以后的版本是会改进的。于是就需要去掉gradle自动编译jni目录了，让我们自己手动编译即可。

在`build.gradle`我们就不用配置ndk节点了，然后在android节点下配置下面信息，修改jniLibs和jni的目录即可。

{% highlight gradle %}

sourceSets.main{
    jniLibs.srcDir 'src/main/libs'
    jni.srcDirs=[]
}

{% endhighlight %}

最后在jni的目录配置就和eclipse的一样了，需要编写Android.mk和Application.mk文件，每次都需要自己在命令行运行ndk-build命令了。

## 4. Application.mk

这里只记录了常用的一些配置，全面详细的解释可以去[官网文档](https://developer.android.com/ndk/guides/application_mk.html)阅读。

先看一个简单的配置：

	APP_ABI := armeabi-v7a
	APP_MODULES := libhelloworld
	APP_PLATFORM := android-19

可以看到里面采用的就是Key-Value的形式来配置的，解释如下：

**APP_ABI**

ABI是Application Binary Interface的意思，即我们需要把代码编译到哪一种cpu架构的指令集。通常包含armeabi，x86，mips等等，更多内容可以阅读[官方文档](https://developer.android.com/ndk/guides/abis.html)。

**APP_MODULES**

它定义编译需要哪些的模块，官方好像已经放弃这个配置了，默认不用配置，ndk-build会寻找Android.mk下的所有模块。

**APP_PLATFORM**

这个变量定义了编译的目标系统平台，貌似不会出现不同版本的不兼容问题，所以填写最新的即可。

**APP_OPTIM**

这个变量定义build出来的是debug还是release，显然debug的话会有更多的信息，而release就会优化速度和库的大小，另外如果Manifest里面定义了`android:debuggable`为true，那么默认这个值就是debug的了。

当我们需要单独使用ndk-build来编译某个库的时候，例如移植libmp3lame的时候，不需要新建一个android项目，只需要把代码放进一个jni目录即可。如果我们在这里运行ndk-build是不行的，会报错找不到项目路径，这个时候我们需要配置下面两个选项。

**APP_PROJECT_PATH**

它定义了项目的路径，实际上是确定了Application.mk的路径，所以这样配置即可：

>APP_PROJECT_PATH :=$(call my-dir)

**APP_BUILD_SCRIPT**

它定义了Android.mk的路径，即编译脚本的位置，这样配置即可：

>APP_BUILD_SCRIPT :=$(call my-dir)/Android.mk

如果没有定义上面两个选项，则在需要这样运行ndk-build命令：

>ndk-build NDK_PROJECT_PATH=. NDK_APPLICATION_MK=Application.mk

## 5. Android.mk

它是GNU Makefile的一个片段，NDK是基于make的，所以本质上它就是makefile，详细可以去看[官网文档](https://developer.android.com/ndk/guides/android_mk.html)。我们可以进入NDK的目录，在build/core目录下有很多的.mk文件，其实就是NDK的构建系统了。mk文件的注释是#号，这个不用多说了，直接看一个简单的例子：

{% highlight shell %}

#当前目录 开头必须以这个开始
LOCAL_PATH := $(call my-dir)

#CLEAR_VARS变量指向了clear-vars.mk位置，include这个可以清除出来LOCAL_PATH以外的LOCAL开头的变量，如LOCAL_MODULE等
#这样做是因为android构建系统再单次执行中解析多个构建文件和模块定义，而LOCAL开头这些是全局变量，清除它们可以避免冲突。
#如果Android.mk文件有多个模块，则每个模块开始之前都要写这行！
include $(CLEAR_VARS)

#库的名字
LOCAL_MODULE    := ZGPlayer

#源码目录
LOCAL_SRC_FILES += \
	src/ZGPlayerJNI.c \
	src/player_main.c

#头文件目录，还可以有各种的编译选项
LOCAL_CFLAGS := \
	-I$(LOCAL_PATH)"/include" \
	-I$(LOCAL_PATH)"/../ffmpeg/include"
	
#连接到一些库
LOCAL_LDLIBS := -llog -ljnigraphics -lz -landroid
LOCAL_SHARED_LIBRARIES := libffmpeg
	
ifeq ($(TARGET_ARCH_ABI),armeabi-v7a)
#C编译器的可选标记选项
	LOCAL_CFLAGS += -DHAVE_NEON=1
#这个会使得去编译.neon文件。
	#LOCAL_ARM_NEON := true
endif

#为了建立供主应用程序使用的模块，必须将该模块编程共享库。这个变量指向了build-shared-library.mk的位置，这个makefie包含了构建的过程。
include $(BUILD_SHARED_LIBRARY)

{% endhighlight %}

基本上也是以Key-Value的形式来定义的，但是支持得比较多，可以累加，还有条件判断，循环，函数等等，实际上它就相当于脚本，每一行都是一个单独的指令。

看到第一行给LOCAL_PATH这个变量赋值，`$()`包含的就是读取变量的值，call是一个指令，my-dir是这个指令的目标，即相当于执行函数。include也是一条指令，它会把外部的mk文件引入进来，例如这里引入了clear-vars.mk文件，然后清空了变量，引入了build-shared-library.mk文件来编译一个共享库。

### 5.1 共享库与静态库

通常编译为so的文件是共享库，它可以直接使用了，也可以提供给别的模块使用，当共享使用的时候需要编写这句`include $(PREBUILT_SHARED_LIBRARY)`，表明它是预编译的，在java里面使用的时候所有的so都需要load的。静态库是.a结尾的文件，它不能直接提供给java使用，但是可以提供给别的模块使用，别的模块会把静态库编译进来只生成一个so库，但是如果一个库要提供给多个模块使用的时候，就不建议用静态块了，那样重复了体积会很大，建议用共享库。静态库的使用如下：

{% highlight shell %}

include $(CLEAR_VARS)
LOCAL_MODULE    := libmp3lame
LOCAL_SRC_FILES := libmp3lame.a
include $(PREBUILT_STATIC_LIBRARY)
//...
LOCAL_WHOLE_STATIC_LIBRARIES += libmp3lame

{% endhighlight %}

### 5.2 LOCAL_CFLAGS编译选项

这个是编译的选项，除了配置头文件目录以外，还可以配置很多信息。而其实配置头文件应该使用`LOCAL_C_INCLUDES`。例子如下：

{% highlight shell %}

#-marm 是指定arm的指令,-mthumb是指定thumb的指令
LOCAL_CFLAGS := -DSTDC_HEADERS

ifeq ($(TARGET_ARCH_ABI), armeabi)
LOCAL_CFLAGS := -marm -mfpu=vfp -mfpu=vfpv3 -DCMP_HAVE_VFP
endif

ifeq ($(TARGET_ARCH_ABI),armeabi-v7a)
# 采用NEON优化技术
LOCAL_ARM_NEON := true
LOCAL_CFLAGS += -DHAVE_NEON=1
endif

{% endhighlight %}

除此之外，gcc一些编译参数都是可以用在这里的，如上面的-fPIC等等。

## 6. 手动加载JNI函数

一般来说，按照上面说的用javah命令生成头文件以后，就会把native方法生成按规则约定的方法名，然后编译器就会自定连接到对应的方法，即加载so库以后，只要调用native方法就会触发调用到了相应的底层c的函数了。新版的JNI系统都是这样自动注册jni函数的了，但是旧版就不一样了，必须根据方法前面注册相应的函数，否则的话就会报一个错误，或者只是警告：

>D/dalvikvm( 2272): No JNI_OnLoad found in /data/data/org.zhangge.jni/lib/libhelloworld.so 0x40517ac8, skipping init

实际上，我们可以理解，jni是把c函数注册到一个列表，当java运行native方法的时候就会去列表里面查找相应的函数，如果找不到的话就抛出无链接的异常，而查找的根据正是方法签名，方法签名的规则是先用括号包起来参数签名，然后是返回值的签名，如果是对象类型需要全名，然后分号隔开。具体每个类型可以看我以前的[虚拟机blog](http://zhgeaits.me/java/2013/04/08/java-classloader-study-notes.html)。

因此，我们需要编写`JNI_OnLoad`函数，它会在加载so的时候调用，然后我们在里面注册相应的native函数即可，同时还要编写`JNI_OnUnload`，来卸载这些函数：

{% highlight c %}

//参数映射表
static JNINativeMethod methods[] = {
        {"dispHelloWorld", "(V)V", Java_org_zhangge_jni_HelloWorld_dispHelloWorld}
};
static int registerNativeMethods(JNIEnv *env, const char *className, JNINativeMethod *gMethods,
                                 int numMethods) {
    jclass clazz;
    clazz = (*env)->FindClass(env, className);
    if (clazz == NULL) {
        return JNI_FALSE;
    }
    if ((*env)->RegisterNatives(env, clazz, gMethods, numMethods) < 0) {
        return JNI_FALSE;
    }
    return JNI_TRUE;
}
static int unregisterNativeMethods(JNIEnv *env, const char *className) {
    jclass clazz;
    clazz = (*env)->FindClass(env, className);
    if (clazz == NULL) {
        return JNI_FALSE;
    }
    if ((*env)->UnregisterNatives(env, clazz) < 0) {
        return JNI_FALSE;
    }
    return JNI_TRUE;
}
static int registerNatives(JNIEnv *env) {
    const char *kClassName = "org/zhangge/jni/HelloWorld";
    return registerNativeMethods(env, kClassName, methods, sizeof(methods) / sizeof(methods[0]));
}
static int unregiterNatives(JNIEnv *env) {
    const char *kClassName = "org/zhangge/jni/HelloWorld";
    return unregisterNativeMethods(env, kClassName);
}
JNIEXPORT jint JNICALL JNI_OnLoad(JavaVM *vm, void *reserved) {
    JNIEnv *env = NULL;
    jint result = -1;
    if ((*vm)->GetEnv(vm, (void **) &env, JNI_VERSION_1_4) != JNI_OK) {
        __android_log_print(ANDROID_LOG_DEBUG, "zhangge", "JNI_OnLoad not version 1.4");
        return -1;
    }
    if (!registerNatives(env)) {
        __android_log_print(ANDROID_LOG_DEBUG, "zhangge", "JNI_OnLoad failed!");
        return -1;
    }
    return JNI_VERSION_1_4;
}
JNIEXPORT void JNICALL JNI_OnUnload(JavaVM *vm, void *reserved) {
    JNIEnv *env = NULL;
    jint result = -1;
    if ((*vm)->GetEnv(vm, (void **) &env, JNI_VERSION_1_4) != JNI_OK) {
        __android_log_print(ANDROID_LOG_DEBUG, "zhangge", "JNI_OnUnload not version 1.4");
        return;
    }
    if (!unregiterNatives(env)) {
        __android_log_print(ANDROID_LOG_DEBUG, "zhangge", "JNI_OnUnload failed!");
        return;
    }
}

{% endhighlight %}

然后其实很多初始化的工作我们可以放到这里来做也是可以的了。

## 7. JNI的相关API使用

主要的API都是JNIEnv这个结构体或者对象了，在2.5有说过一些JNIEnv的东西，其实就是定义在`jni.h`的结构体，里面含有大量的函数指针。

### 7.1 JNI回调Java方法

其实用到的是反射，看下面如何获取Android的Context对象：

{% highlight c %}

/**
 * 获取context
 * 坑爹的一个问题:在一个2.3的三星手机上,在播放时获取不到context,估计是因为ActivityThread被占用所以无法反射
 */
jobject getGlobalContext(JNIEnv *env) {
    jclass activityThread = (*env)->FindClass(env, "android/app/ActivityThread");
    jmethodID currentActivityThread = (*env)->GetStaticMethodID(env, activityThread,
                                                                "currentActivityThread",
                                                                "()Landroid/app/ActivityThread;");
    jobject at = (*env)->CallStaticObjectMethod(env, activityThread, currentActivityThread);
    jmethodID getApplication = (*env)->GetMethodID(env, activityThread, "getApplication",
                                                   "()Landroid/app/Application;");
    jobject context = (*env)->CallObjectMethod(env, at, getApplication);
    return context;
}

{% endhighlight %}

反射的开销还是有的，不要频繁调用，可以保存起来一些信息。注意到要认真看JNIEnv的各种方法，除了CallStaticObjectMethod，还有CallVoidMethod等等！

### 7.2 java数组转c数组

{% highlight c %}

//传进来的buffer和length
jbyteArray buffer;
char* data = (char*)(*env)->GetByteArrayElements(env, buffer, 0);
//do something with data
(*env)->SetByteArrayRegion(env, buffer, 0, length, data);
//这里有个坑,如果不释放的话就会报错
//Failed adding to JNI pinned array ref table (1024 entries)
//这不是释放byte数组,而是释放它的索引,它的索引存在一个table里面,低端机table比较小
(*env)->ReleaseByteArrayElements(env, buffer, data, 0);

{% endhighlight %}

真的要时刻调用ReleaseByteArrayElements类似的方法，它不是释放数组的内容，而是释放引用，在jni里面，所有的java引用都存放在一个table里面，这个table很小的，一旦满了就会崩溃，不会自动去释放的。

### 7.3 打印日志

需要`include <android/log.h>`，然后就可以使用接口了：

>__android_log_print(ANDROID_LOG_DEBUG, "zhangge-test", "code format here:%s", "hahaha");

同样我们可以定义在宏那里，使用更方面，它直接支持不定参数的传递，比写方法方便多了，如果要看C的不定参数，可以去看另外一篇[blog](http://zhgeaits.me/c/2013/06/09/c-varargs.html)。

`#define LOGE(format, ...) __android_log_print(ANDROID_LOG_ERROR, "zhangge-test", format, ##__VA_ARGS__)`	

## 8. so加解密

以前想过把核心代码写到so别人就很难破解了，特别是我做3D播放器的时候，想要把一些OpenGLES的代码写到so里面去，就是一些字符串而已，当我全都写好了以后，无意中一次在Eclipse里面用Text文本器打开了so，居然全部字符串的东西都能看到了！瞬间就被打击惨了。另外就算如果把代码写到so里面，那么native方法是不可以被混淆的，别人很容易直接拿你的库来使用。

对于字符串的问题，解决方法是，先声明一个字符数组，然后在初始化函数里面一个字符一个字符的来赋值，那么就不会那么容易被破解了。如果要防止别人调用自己的so，可以在运行的时候取获取Android的Context，然后读取包名和签名的hash值，然后比较，存包名和hash值的时候不能直接原来的值直接存，要分段存储，然后还可以异或算法加密，校验的时候拼接以后解密再比较即可。不能通过jni调用java方法还验证，那样会抛出异常告诉用户具体哪个方法没找到，根本没有作用的。因为jni不能捕获异常啊！至于其他高级的方法就有待深入研究了，那需要修改到汇编的代码了。

另外关于C和CPP的语言相关知识就不记录在这里了。