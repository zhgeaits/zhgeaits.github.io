---
layout: post
title:  "Android OpenGLES学习之入门篇"
date:   2014-10-16 10:06:03
categories: android
type: android
---

## 1 引子

### 1.1 理解图形编程

我们知道计算机的发展历史，了解过比尔盖茨和乔布斯的故事，清楚知道图形发展的重要性，苹果电脑的，windows系统，今天各种绚丽的画面，游戏，等等。我们也知道组装电脑的时候，要选一块好的显卡，才能玩更好的游戏。而实际上，我们对计算机的了解并不多。

当我们学习编程的时候，盯着控制台，看到输出“Hello world！”，会为这一伟大的成就欢呼。然而，这不过是几十年前的技术罢了；现在问题来了，我们只会在控制台输出文字，那到底怎么样编写图形界面呢？

不管是学习哪一门语言，c，java，实际上我们都是针对cpu来编程，所有的逻辑计算都是由CPU来计算的，这个我们学习过计算机组成原理，我们知道都是cpu的指令集在操作寄存器而已。对于输出的结果显示，基本上都是文字字母，简单的在阴极射线管显示器即可显示。对CPU的操作和对输出结果的显示，这一切都是操作系统和编译器完成的事情，把01指令的机器语言封装成汇编语言，再往上是高级语言，我们只需要调用系统接口就可以了，而其实系统对大部分硬件的操作都是通过驱动程序完成的，每一个硬件都有对应的驱动程序，驱动程序要遵循统一的规范和接口，然后实现即可。所以每次我们重装完系统，都要安装各种各样的驱动程序。对于硬件的开发，是嵌入式的领域了，虽然不用深入探讨，但是从这里可以理解到，硬件也是有处理单元的！

那终于知道其实显卡是有图形处理单元的，也就是GPU（Graphic Process Unit），和CPU一样的意义，用于渲染画面。那么问题又来了，图形怎么绘制的？其实我们都知道，那就是像素，我们现在是电子设备爆炸的年代，谁不知道分辨率这东西呢？一个显示器的分辨率是1920x1080，意思就是横向有1920个像素，纵向有1080个像素，我们可以理解，像素就是一个很小很小的发光点，所以说，屏幕其实是汇聚了超密集的像素点，当然，这会涉及到材料学领域了，如液晶屏等等。我们只要抽象来学习理解即可。学过物理，我们也知道三原色，只要红绿蓝即可调出所有颜色，也就是说，其实每个像素点都是有不同量的红绿蓝三色。当然啦，显示器肯定不是那么简单，原生的数据是yuv，这又涉及到深入图形学领域了，我们现在只要抽象理解即可。

一般红绿蓝的取值范围是0-255，也就是一个8位的字节，再加上一个透明度alpha，rgba刚好是四个字节，通常是用一个整形来表示。现在我们就能理解到，实际上一张二维的图片，就是一个二维整形矩阵，这些都是我们在cpu和内存都可以操作的逻辑了。最后操作系统把数据交换给显卡，通常是复制到显存，交给了GPU来渲染显示即完成了。

### 1.2 图形编程API

通过上面的历史了解，我们十分清晰，图形领域是十分重要和可发展的，于是，介于操作系统和硬件（驱动）的中间层，可以做很多事情，提供重要的图形编程接口，方便开发二维和三维的图形。于是，这个世界基本上又出现了两大阵型，OpenGL和DirectX。

DirectX太熟悉了，我们打游戏，一定要安装这个东西，童年啊！它就是用于windows的游戏开发，一统天下！而OpenGL是跨平台的，不管哪里都能用！并且不仅用于游戏开发，几乎什么领域都可以用。当然了，很少人会直接用这两个东西直接开发游戏，而是用来开发游戏引擎，然后基于引擎开发游戏，例如unity！

opengl的全称是Open Graphics Library，关于OpenGL的开发，我们用到的是着色语言（shading language），更多的历史信息介绍前往百科即可。

## 2 什么是OpenGL ES

由于移动设备的快速发展，于是出现了针对这些嵌入式设备的一套API子集出台了，OpenGL for Embedded Systems。显然，因为是子集，所以就是对于OpenGL进行了功能的裁剪。然后OpenGL ES现在已经发展到了3.0版本，每一个版本都是巨大的飞跃，我这里主要学习的是2.0版本。1.0和2.0之间的巨大区别在于，2.0是支持自定义渲染管线编程的，也就是可以支持着色语言了，意思就是，我们除了在高级语言那里调用opengl的api，还能在管线里面编写着色语言程序！

### 2.1 OpenGL ES 1.0的渲染管线

GPU内部有许多处理图形信号的并行处理单元，所以它比CPU的串行执行效率高很多。渲染管线实际上就是通过处理单元处理的一系列绘制过程。管线如下图所示：

![alt opengles1.0pineline](/image/opengles1.0pineline.png "opengles1.0pineline")

重点理解几点：
  
- 什么是图元，其实就是图像单元；绘制方式有点，线和三角形，分别对应三种图元。
- 什么是光栅化，图元在数学上是连续的量，但是在显示器就是离散的像素，所以，光栅化就是把顶点数据转换为片元的过程。
- 什么是片元，为什么不叫像素？像素是屏幕上的点，那是二维的，但是一个屏幕上的像素在三维中，可能覆盖了很多个像素，于是在三维中不能叫像素，应该叫片元。

### 2.2 OpenGL ES 2.0的渲染管线

2.0的管线如下图所示：

![alt opengles2.0pineline](/image/opengles2.0pineline.png "opengles2.0pineline")

2.0的最大区别就是多了顶点着色器和片元着色器。

#### 2.2.1 定点着色器

什么是顶点？就是几何图形的顶点的意思，例如三角形有3个顶点，矩形有4个顶点，线段只有两个顶点。顶点着色器是就是一个可编程的处理单元，图形的每一个顶点都会经过顶点着色器进行处理转换，产生纹理坐标，颜色，点位置等所需的顶点属性信息。工作原理图如下：

![alt opengles2.0vertexshader](/image/opengles2.0vertexshader.png "opengles2.0vertexshader")

其中attribute是每个顶点各自不同信息所属的变量，一般包含顶点坐标和顶点纹理坐标，通过高级语言传输下来的；uniform是全局变量，是通过高级语言传输下来的数据；varying是产生输出到片元着色器的数据，一般是纹理坐标。

一个顶点着色器例子代码如下：

{% highlight bash %}

uniform mat4 uMVPMatrix; //总变换矩阵
attribute vec3 aPosition;  //顶点位置
attribute vec2 aTexCoor;    //顶点纹理坐标
varying vec2 vTextureCoord;  //用于传递给片元着色器的变量
void main()     
{                            		
   gl_Position = uMVPMatrix * vec4(aPosition,1); //根据总变换矩阵计算此次绘制此顶点位置
   vTextureCoord = aTexCoor;//将接收的纹理坐标传递给片元着色器
}

{% endhighlight %}

#### 2.2.2 片元着色器

光栅化的每个片元都会执行一次片元着色器，可以理解为每个像素都执行一次（二维），主要的功能是纹理的采样，颜色的汇总。

什么是纹理，直接理解为就是图形的表面，皮肤之类，也就是图像，颜色，花纹等等。例如，在android中，把一张图片bitmap直接映射到opengle中成为一张纹理，这时候纹理就是一张图片了，bitmap是可以回收的了。

工作原理图如下：

![alt opengles2.0fragmentshader](/image/opengles2.0fragmentshader.png "opengles2.0fragmentshader")

varying数据是从顶点着色器来的；uniform通常是纹理的数据。

一个片元着色器例子代码如下：

{% highlight bash %}

precision mediump float;
varying vec2 vTextureCoord; //接收从顶点着色器过来的参数
uniform sampler2D sTexture;//纹理内容数据
void main()                         
{           
   //给此片元从纹理中采样出颜色值            
   gl_FragColor = texture2D(sTexture, vTextureCoord); 
}

{% endhighlight %}

>实际上，绘制一个矩形是通过绘制两个三角形合成的，也就是有6个顶点，每个顶点执行一次顶点着色器，然而顶点着色器输出的用于传递给片元着色器的坐标变量并没有直接传递给片元着色器，而是在光栅化以后，通过插值计算，得出每个片元的坐标再传递给片元着色器，于是，片元着色器是执行处理每一个片元（像素）的。例如，把一张图片绘制满1920x1080的屏幕，则每个像素都执行一遍片元着色器。

### 2.3 着色语言

看到这里，基本上已经能够理解OpenGLES的原理了，着色语言就是又是一大块知识，学习它就能在上面两个着色器上编写代码了，这里不可能完全列举出所有知识，网上有很多着色语言的资料，学习这个就跟学习一门语言差不多，而且不难理解。

简单知道几个向量类型，mat4是四维矩阵，vec4是4维向量，vec3是3维向量，vec2是2维向量。

## 3 Android上开发OpenGLES

实际上，还是会有很多基本的东西没有说到的，而且难于理解，这里先通过一个简单的例子告诉如何在android上开发opengles程序。后面会通过一些实际的应用来解析更多。我们使用的opengles2.0来实现，通过上面的知识，我们知道实现的思路是，在android平台上，用高级语言java来调用opengles的api，把信息和数据传输至opengles的管线，当然这个api是android系统提供的，主要还是native的接口，然后再结合编写着色语言来绘制顶点和片元，最后由opengles来渲染显示在屏幕上。

OK，那么我们就来绘制一个三角形。

### 3.1 创建项目

打开android studio，新建一个项目，由于opengles是没有sdk版本的限制，所以随意创建一个空项目即可。

### 3.2 编写着色器代码

在assets目录新建vertex.sh和frag.sh，分别编写着色器代码，如下：

{% highlight bash %}

uniform mat4 uMVPMatrix; //总变换矩阵
attribute vec3 aPosition;  //顶点位置
attribute vec4 aColor;    //顶点颜色
varying  vec4 vColor;  //用于传递给片元着色器的变量

void main()
{
   gl_Position = uMVPMatrix * vec4(aPosition,1); //根据总变换矩阵计算此次绘制此顶点位置
   vColor = aColor;//将接收的颜色传递给片元着色器
}

{% endhighlight %}

由前面的理论可以知道，aPosition和aColor都是java层传递下来的坐标，而vColor是输出到下一个流程的，实际上是经过光栅化，然后插值成每个片元，再传递到片元着色器的。所以实际上，上面这段代码执行了三遍，因为三角形只有三个顶点。变换矩阵暂时不用理，通过矩阵变换得到顶点的最终坐标，然后赋值给了gl_Position。

{% highlight bash %}

precision mediump float;
varying  vec4 vColor; //接收从顶点着色器过来的参数

void main()                         
{                       
   gl_FragColor = vColor;//给此片元颜色值
}

{% endhighlight %}

通过前面的理论可以知道每个片元都执行这段代码，可以知道实际上颜色是渐变的，因为插值。也就是说，其实java程序设定了三个顶点的颜色，然后通过光栅化插值来填充了三角形的内部颜色。

### 3.3 创建一个工具类来加载着色器代码

基本的步骤是，首先创建着色器shader，然后加载着色器代码（vertex.sh/frag.sh），再编译这些代码，最后创建一个gl程序，attach这些编译的代码，最终链接程序即可。这里就不上代码了

### 3.4 创建绘制的View

首先，要知道绘制OpenGL是通过GLSurfaceView，显然，它是继承于SurfaceView的，关于SurfaceView，可以从另外一篇blog知道。我们创建一个MyOpenGLESView，继承GLSurfaceView；然后还要创建一个内部类MyOpenGLESRenderer，它实现了GLSurfaceView.Renderer的接口。

代码如下：

{% highlight java %}

public class MyOpenGLESView extends GLSurfaceView {

    private MyOpenGLESRenderer mRenderer;

    public MyOpenGLESView(Context context) {
        super(context);
        this.setEGLContextClientVersion(2);
        mRenderer = new MyOpenGLESRenderer();
        this.setRenderer(mRenderer);
        this.setRenderMode(GLSurfaceView.RENDERMODE_CONTINUOUSLY);
    }

    private class MyOpenGLESRenderer implements GLSurfaceView.Renderer {
        private Triangle tle;

        public void onDrawFrame(GL10 gl) {
            //清除深度缓冲与颜色缓冲
            GLES20.glClear(GLES20.GL_DEPTH_BUFFER_BIT | GLES20.GL_COLOR_BUFFER_BIT);
            //绘制三角形
            tle.drawSelf();
        }

        public void onSurfaceChanged(GL10 gl, int width, int height) {
            //...
        }

        public void onSurfaceCreated(GL10 gl, EGLConfig config) {
            //设置屏幕背景色RGBA
            GLES20.glClearColor(0, 0, 0, 1.0f);
            //创建三角形对对象
            tle = new Triangle(MyOpenGLESView.this);
            //打开深度检测
            GLES20.glEnable(GLES20.GL_DEPTH_TEST);
        }
    }
}

{% endhighlight %}

可以看到的是创建一个渲染器，然后丢给glsurfaceview，然后在ondrawframe里面进行绘制。
实际上是有一条opengles的线程，里面有它的context，它会持续的回调onDrawFrame方法进行绘制的。

### 3.5 创建一个三角类

实际的绘制逻辑是在三角形类里面的。首先要初始化三角形的顶点坐标数据和顶点颜色数据，然后初始化着色器。然后在drawSelf方法里面使用着色器程序和传递顶点数据下去即可。

### 3.6 在界面上显示

在Activity里面直接使用MyOpenGLESView即可，注意的是，在onResume和onPause方法分别调用，本质是wait和wake渲染线程。

代码和效果就不上了，因为我不建议复制粘贴，直接去github上clone下来，跑起来，动手修改看看吧。[传送门](https://github.com/zhgeaits/AndroidOpenGLESDemo)

另外，demo代码还是很多没有解析的，那些将在后面的blog介绍，关注下一篇，[Android OpenGLES学习之实现弹幕渲染]()。