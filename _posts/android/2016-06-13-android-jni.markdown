---
layout: post
title:  "Android JNI Develop Stuff"
date:   2016-06-13 11:00:03
categories: android
supertype: career
type: android
---

>这里必须先说一个故事，在我大三拿到offer以后决定转行Android，然后决定在大四的时候一变游玩一边写一个Android项目，以备毕业入职之需。然而，在我还没入门的时候就拿到工作室的高大上的项目，做一个裸眼3D播放器，哈哈！我还没熟悉Android的时候就涉足了JNI的开发，那个时候主要涉及的都是底层的C和视频编解码相关的知识！就是没有记录多少在Blog上。那时候用的还是Eclipse，现在工作两年了，早已放弃了用Android Studio了，现在AS对JNI的支持还是不太好，刚好现在的工作再次涉及到JNI了，所以便有了这篇...

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

命名：lib是必须的，helloworld是loadLibrary指定的。

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

这个extern "C"包含了我们的函数的原型定义。原本extern关键字是修饰变量的，使得我们可以获取外部的变量，而这里结合了“C”来使用，告诉编译器这些函数按照C的方式来编译，那就不会出现错误了。关于extern这一块的知识，还是蛮多的，因为我不做c++开发，所以没有深入理解。

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

## 3. Android JNI开发

在Android上也是可以进行jni开发的，因为android也是基于linux系统来的，也是可以在dvm里面跑c的代码，不过需要用到ndk里面的`ndk-build`来编译，并不是说随便拿一个so库来就可以执行的，必须通过ndk编译过的才能在android系统上运行，具体估计跟编译环境和系统等等有关系，我还没学习过相关知识，所以无法解释了。[ndk-build](https://developer.android.com/ndk/guides/ndk-build.html)是Google提供的一个编译脚本。

__为什么要去写native代码？__

首先是需要实现java无法实现的功能，有些时候可能是java没有api支持，那么我们如果对c比较熟悉的话就可以去写c来实现了；其次，当我们需要提高性能的时候，用c写的效率是会比较高的，就是如果要实现性能优化的时候可以考虑jni开发；如果开发的功能依赖到第三方的库来实现，如ffmpeg，shine, sdl等等，这些都是c来实现的，那么我们就需要移植到android，然后自己再用jni进行开发了。再就是想要动态加载库的时候，我们可以把核心模块写到c里面，不需要集成到apk里面去，当运行到这里的时候才让用户进行下载so库加载（jar包可以么？）；还有，如果我们想要保护核心的功能不被破解，是可以把功能写到so里面的，破解so至少比破解jar难一些，不过也是可以的，对于安全这块又是一大块领域了。最后，因为开发的so是native代码实现的，几乎是可以在其他平台上重用的！

关于一些ndk的概念，可以从[官网](https://developer.android.com/ndk/guides/concepts.html#hiw)上阅读更多。

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

全局jni目录下只有一个`Application.mk`，这个文件列举和描述了我们app所需要的模块，里面主要包括了编译的abi，toolchains和包含的标准库(static、dynamic STLport 或者 default system)。

我们至少需要一个`Android.mk`文件，当有多个模块的时候就需要多个Android.mk文件了。里面定义了模块和它的名字，需要编译的文件，build flags和需要链接的库，每次运行ndk-build脚本命令的时候必然会去寻找这个文件进行编译的。通常有些变量可以在Application.mk全局定义，会应用到每一个Android.mk。

### 3.3 Android Studio配置

在as2.0以后对jni的支持已经相对够好了，想SDK一样配置NDK的目录即可，结合gradle基本上能完成基本的开发，但是对于第三方库的时候，却很多地方无法配置，无奈我只能去掉jni的支持，自己手动执行ndk-build来编译，因为使用了gradle，所以一般来说，是不需要Android.mk和Application.mk配置文件的了。

#### 3.3.1 添加JNI支持

只需要在main目录下，右键选择new，在Folder那里选择Add Jni Folder即可，其实直接new一个名字为jni的目录也是可以的。gradle在构建的时候就会去检查并执行ndk命令的了。

然后把javah生成的头文件复制到jni目录，在去编写c文件就可以的了。生成的so库不在install到外面的libs目录，libs目录一般放的是第三方的jar包了。它会install到了build目录intermediates/ndk下的了。

然后在`build.gradle`文件的defaultConfig节点下加入下面配置：

{% highlight gradle %}

ndk{
    moduleName "helloworld"
    abiFilters "armeabi", "armeabi-v7a", "x86", "mips"
    ldLibs "log"
}

{% endhighlight %}

可以看到，其实配置的内容和两个.mk文件是一样的，上面的字段都比较容易理解，点进去还能看到更多的信息。

运行的时候应该会报错的，as对jni的支持还不够好，我们在`gradle.properties`文件加上这一行即可：

>android.useDeprecatedNdk=true

#### 3.3.2 去掉jni的支持

在我要使用第三方的so库，可以是复制到libs目录，这是不合理的用法，另外，如果使用的是.a静态库并要加入到jni的编译中去的时候就没法做了。因为很多Android.mk的配置，在gradle上还没有支持，以后的版本是会改进的。于是就需要去掉gradle自动编译jni目录了，让我们自己手动编译即可。

在`build.gradle`我们就不用配置ndk节点了，然后在android节点下配置下面信息，修改jniLibs和jni的目录即可。

{% highlight gradle %}

sourceSets.main{
    jniLibs.srcDir 'src/main/libs'
    jni.srcDirs=[]
}

{% endhighlight %}

最后在jni的目录配置就和eclipse的一样了，需要编写Android.mk和Application.mk文件，每次都需要自己在命令行运行ndk-build命令了。

### 4. Application.mk

这里只记录了常用的一些配置，全面详细的解释可以去[官网文档](https://developer.android.com/ndk/guides/application_mk.html)阅读。

先看一个简单的配置：

	APP_ABI := armeabi-v7a
	APP_MODULES := libhelloworld
	APP_PLATFORM := android-19

可以看到里面采用的就是Key-Value的形式来配置的，解释如下：

**APP_AB**

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

### 5. Android.mk

[官网文档](https://developer.android.com/ndk/guides/android_mk.html)

### 6. 手动加载JNI函数

### 7. JNI的相关API使用

### 8. so加解密

另外关于C和CPP的语言相关知识就不记录在这里了。