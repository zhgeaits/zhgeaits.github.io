---
layout: post
title:  "Android OpenGLES学习之实现弹幕渲染"
date:   2016-02-27 11:00:03
categories: android
type: android
---

>代码实现已经开源，[猛戳这里](https://github.com/zhgeaits/ZGDanmaku)。

## 1 弹幕的原理

这里我们的重点是使用OpenGLES实现渲染，而不是弹幕的逻辑优化，所以，只需要用到最简单的弹幕实现逻辑即可。

想一想各大网站上的弹幕效果，在视频之上弹幕文字从右到左出现，然后滚动，一直到左边屏幕之外然后消失。新手首先想到的可能会是把一个个的TextView设置一个从左往右的动画来实现，虽然效果是有的，但是想想就会知道，成千上万个弹幕飘过，那得多少个动画？要多少个TextView？性能会是一个大问题，再说，实现逻辑上也会有一定的问题。

学习过[Android的View绘制原理]()，我们就能想到另外一个思路来实现。

>在屏幕上放一张“画布”，然后把一个个的弹幕画在上面，起始位置是右边屏幕之外，随着时间的变化，时间差乘以弹幕的速度，即可得出弹幕移动的距离。

画布的实现就是一个View，当然也可以用SurfaceView或者TextureView来实现。我们首先把弹幕绘制在bitmap里面，然后只需要实现一个DanmakuView即可，它继承于View，然后我们把弹幕都绘制在这个View上面；我们知道View有一个方法：

>protected void onDraw(Canvas canvas);

拿到canvas就可以把bitmap绘制在上面了。我们可以开一个线程每帧都刷新一遍这个view就实现了弹幕。另外一点我们要清楚的是，android的坐标系是从左上角开始，这样我们计算移动距离就不会出错了。其他弹幕的逻辑就不详细说了。

## 2 OpenGLES的原理

从这篇[blog](http://zhgeaits.me/android/2014/10/16/android-opengles-study-notes.html)里我们理解了OpenGLES的机制原理，并且通过一个绘制三角形的例子学习了如果在android平台上调用OpenGLES的API进行绘制渲染。简而言之，就是把顶点的坐标和颜色传递给渲染管线，并且我们可以在管线的顶点着色器和片元着色器编写代码进行对图像的处理，最终输出在屏幕上。

这里继续学习更多需要用到的知识：

### 2.1 纹理

之前提到纹理就是物体的表面，形象上面是可以这样理解，更具体一点可以理解就是一个贴图，例如，我要渲染一张图片，那么就先要把这张图片映射到纹理之上，然后OpenGLES就可以把这张问题渲染出来了。本质上，可以看出来，纹理就是一块显存，相当于bitmap就是一块内存，但是还是有很大的区别的。毕竟现在是初学，暂时这么肤浅的理解。

生成纹理代码：

{% highlight java %}

//生成纹理ID
int[] textures = new int[1];

//第一个参数是生成纹理的数量，最后一个参数是偏移量
GLES20.glGenTextures(1, textures, 0);

{% endhighlight %}

学习过c，unix，我们很容易理解什么是文件描述符，也清楚为什么是一个整型表示的；同理，在OpenGLES这里，基本上所有的“引用”都是用整型ID来表示的，直接理解为java的引用即可。拿到纹理id就相当于拿到了纹理，注意到的是，它不会像bitmap那样需要指定大小。而其实如果生成大量的纹理也是会出现oom的！

### 2.2 坐标系

#### 2.2.1 android系统原生的坐标系

这我们都知道，android屏幕是二维的，左上角是原点(0,0)，然后往下是y轴递增，往右是x轴递增，范围就是屏幕的像素，例如一个1080p的手机屏幕，横屏的时候，右上角的坐标是(1920, 0)，左下角的坐标是(0, 1080)，右下角的坐标是(1920, 1080)。不用画图了，想象一下就知道了。

#### 2.2.2 纹理坐标

纹理坐标对于android原生的坐标系做了归一化处理，学过数学的都知道，小学就知道什么是“单位一”了，大学过了也该知道归一化。对于一张bitmap，假设大小是360x480，那么它的坐标如下：

![alt texturecoord2](/image/texturecoord2.png "texturecoord2")

那么做了归一化处理以后，实际上，坐标系就是0-1的正方形了，坐标系如下：

![alt texturecoord1](/image/texturecoord1.png "texturecoord1")

为什么要归一化处理，呃，方便显卡计算！至于两个坐标之间的转换更不用说了！

#### 2.2.3 顶点坐标系

从bitmap的坐标归一化到纹理坐标还是比较好理解的，但是从屏幕坐标归一化到顶点坐标就变化很大了。这时候屏幕的中点是原点，取值范围不是0-1了，而是-1到1，如下图所示：

![alt vertexcoord](/image/vertexcoord.png "vertexcoord")

这就是我们中学的时候学习的坐标系，不过范围不同而已；另外，这里我去掉了z轴，因为目前我只讨论二维，使z=0便罢。需要注意的是，当我们把屏幕坐标归一化为顶点坐标的计算的时候，结果要放大两倍，因为不再是一个单位了，而是两个单位！

### 2.3 矩阵变换

很容易想到，如果要让物体移动，例如平移，就直接修改x坐标即可。除了平移，还有旋转，缩放。这些的变换都是可以通过矩阵乘法实现的，也就是矩阵乘以向量。线性代数的最基本知识了，没学高数怪你咯！每个顶点坐标都是一个四维向量(x, y, z, 1)；所以矩阵也是一个四维矩阵，我们要对顶点进行变换，实际上就是对矩阵进行操作，最后把矩阵传递到顶点着色器，然后相乘就得到了最终的坐标了。那现在就知道顶点着色器里面的变换矩阵是干嘛用的了。例如下面对x，y，z分别进行平移v1, v2, v3的距离：

![alt openglesjuzhenpingyi](/image/openglesjuzhenpingyi.png "openglesjuzhenpingyi")

## 3 弹幕的实现

到这里基本上所需要的理论都具备了，应该能想到实现的思路了：

>把弹幕bitmap映射到纹理Texture，然后计算各种坐标，变换矩阵，最后把纹理交给管线去渲染。随着时间的变化，弹幕的x轴是不断变化的，也就是我们不断计算变换矩阵即可。

### 3.1 编写着色器

顶点着色器接收顶点的初始位置坐标，纹理坐标，变换矩阵；用变换矩阵计算顶点的偏移后坐标；把纹理坐标直接传递给片元着色器，代码如下：

{% highlight bash %}

uniform mat4 uMVPMatrix; //总变换矩阵
attribute vec3 aPosition;  //顶点位置
attribute vec2 aTexCoor;    //纹理坐标
varying vec2 vTextureCoord;  //用于传递给片元着色器的变量
void main()     
{                            		
   gl_Position = uMVPMatrix * vec4(aPosition,1); //根据总变换矩阵计算此次绘制此顶点位置
   vTextureCoord = aTexCoor;//将接收的纹理坐标传递给片元着色器
}

{% endhighlight %}

实际上片元着色器接收到的是光栅化插值以后的纹理坐标，然后我们根据这个坐标去纹理那里采样颜色即可。代码如下：

{% highlight bash %}

precision mediump float;//告诉精度是float
varying vec2 vTextureCoord; //接收的纹理坐标
uniform sampler2D sTexture;//纹理内容数据
void main()                         
{           
   //给此片元从纹理中采样出颜色值            
   gl_FragColor = texture2D(sTexture, vTextureCoord); 
}

{% endhighlight %}

着色器代码放在assets，在demo那已经讲过了加载的流程，这里不再重复了。

### 3.2 编写DanmakuView类和渲染器ZGDanmakuRenderer

继承于GLSurfaceView，关于这点不再详述，要注意的是`setEGLContextClientVersion()`方法必须首先执行，不然会出错的。另外，什么是EGL，是介于诸如OpenGL或OpenVG的Khronos渲染API与底层本地平台窗口系统的接口，被用于处理图形管理、表面/缓冲捆绑、渲染同步及支援使用其他Khronos API进行的高效、加速、混合模式2D和3D渲染。好吧，这也不是一两句能解释清楚的！反正它就是android的一个框架层，负责加载OpenGLES函数库和本地实现。

{% highlight java %}

public class ZGDanmakuView extends GLSurfaceView {
    private Context mContext;
    public ZGDanmakuView(Context context) {
        super(context);
        init(context);
    }
    public ZGDanmakuView(Context context, AttributeSet attrs) {
        super(context, attrs);
        init(context);
    }
    private void init(Context context) {
        this.mContext = context;

        //设置使用opengles 2.0
        setEGLContextClientVersion(2);

        //设置EGL的像素配置
        setEGLConfigChooser(8, 8, 8, 8, 16, 0);

        ZGDanmakuRenderer renderer = new ZGDanmakuRenderer();
        setRenderer(renderer);

        //设置view为透明，并置于顶层，可以在surfaceview之上
        getHolder().setFormat(PixelFormat.TRANSLUCENT);
        setZOrderOnTop(true);

        //设置渲染模式为主动渲染，即有一条后台OPENGL线程每帧都调用onDrawFrame方法
        setRenderMode(GLSurfaceView.RENDERMODE_CONTINUOUSLY);
    }
}

{% endhighlight %}

渲染器ZGDanmakuRenderer代码如下，它其实就是渲染的回调罢了，懂surfaceview就好理解了。

{% highlight java %}

public class ZGDanmakuRenderer implements GLSurfaceView.Renderer {
	//...省略不必要的代码了
    @Override
    public void onSurfaceChanged(GL10 gl10, int width, int height) {
        this.mViewWidth = width;
        this.mViewHeight = height;

        //设置视窗大小及位置为整个view范围
        GLES20.glViewport(0, 0, width, height);

        //计算产生正交投影矩阵
        //一般会设置前两个参数为-width / height，width / height，使得纹理不会变形，
        //但是我这里不这样设置，为了控制位置，变形这个问题在顶点坐标那里处理即可
        MatrixUtils.setProjectOrtho(-1, 1, -1, 1, 0, 1);

        //产生摄像机9参数位置矩阵
        MatrixUtils.setCamera(0, 0, 1, 0f, 0f, 0f, 0f, 1, 0);

        mLastTime = SystemClock.elapsedRealtime();
    }

    @Override
    public void onDrawFrame(GL10 gl10) {
        long currentTime = SystemClock.elapsedRealtime();
        float intervalTime = (float)(currentTime - mLastTime) / 1000.0f;
        float detalOffset = mSpeed * intervalTime;

        //设置屏幕背景色RGBA
        GLES20.glClearColor(0.0f, 0.0f, 0.0f, 0.0f);

        //清除深度缓冲与颜色缓冲
        GLES20.glClear(GLES20.GL_DEPTH_BUFFER_BIT | GLES20.GL_COLOR_BUFFER_BIT);

        //绘制弹幕纹理
        List<ZGDanmaku> danmakus = mDanmakus;
        int size = danmakus.size();
        for (int i = 0; i < size; i ++) {
            ZGDanmaku danmaku = danmakus.get(i);
            float newOffset = detalOffset + danmaku.getCurrentOffsetX();
            danmaku.setOffsetX(newOffset);
            danmaku.drawDanmaku();
        }
        mLastTime = currentTime;
    }
}

{% endhighlight %}

可以看到的是，每帧渲染时都会清屏，然后计算弹幕的新位置，再次绘制。

### 3.3 实现ZGDanmaku类

主要的逻辑都是在这个类里面的，现在拆分每一部分进行解释。

#### 3.3.1 初始化顶点坐标与纹理坐标

从2.2中我们已经了解了OpenGLES的坐标系，又由3.1知道我们需要归一化计算弹幕的顶点坐标和纹理坐标，从纹理坐标来采样纹理的颜色，对于纹理坐标，我设置的是全部，即完整的绘制纹理；代码如下：

{% highlight java %}

public void initVertexData() {
    //顶点坐标数据
    //首先归一化计算弹幕在坐标系中的宽高
    float danmakuHeight = (float) mBitmap.getHeight() / mViewHeight * 2.0f;
    float danmakuWidth = (float) mBitmap.getWidth() / mViewWidth * 2.0f;

    //弹幕四个角的顶点坐标，我默认把它绘制在屏幕的左上角了，这样方便理解偏移计算
    //为什么不是顺时针或者逆时针，实际上opengl只能绘制三角形，所以，
    //这里其实是绘制了两个三角形的，前三个点和后三个点分别是三角形
    float vertices[] = new float[]
            {
                    -1, 1, 0,                                   //左上角
                    -(1 - danmakuWidth), 1, 0,                  //右上角
                    -1, 1 - danmakuHeight, 0,                   //左下角
                    -(1 - danmakuWidth), 1 - danmakuHeight, 0   //右下角
            };

    //一个float是4个字节，所以*4
    //关于ByteBuffer，可以看nio相关知识：http://zhgeaits.me/java/2013/06/17/Java-nio.html
    ByteBuffer vbb = ByteBuffer.allocateDirect(vertices.length * 4);
    vbb.order(ByteOrder.nativeOrder());
    mVertexBuffer = vbb.asFloatBuffer();
    mVertexBuffer.put(vertices);
    mVertexBuffer.position(0);

    //纹理坐标，同理是两个三角形的图元。
    float texCoor[] = new float[]
            {
                    0, 0,   //左上角
                    1, 0,   //右上角
                    0, 1,   //左下角
                    1, 1    //右下角
            };

    ByteBuffer cbb = ByteBuffer.allocateDirect(texCoor.length * 4);
    cbb.order(ByteOrder.nativeOrder());
    mTexCoorBuffer = cbb.asFloatBuffer();
    mTexCoorBuffer.put(texCoor);
    mTexCoorBuffer.position(0);
}

{% endhighlight %}

>扩展：纹理是0-1的坐标系，可以理解到这点，假如我们给的纹理坐标不是4个点，而是一个三角形，那么我们最终绘制的是裁剪过后的三角形。

对于顶点坐标，效果是如何的，看下图就可以明白了：

![alt openglesdanmakuvertex](/image/openglesdanmakuvertex.png "openglesdanmakuvertex")

关于三角形绘制，在学习渲染管线的时候解释过图元和绘制方式，我们这里采用三角形的绘制方式，`GLES20.GL_TRIANGLE_STRIP`，因此一个弹幕是一个矩形，一个矩形由两个三角形组成，即两个图元，如下图所示：

![alt openglessanjiaoxing](/image/openglessanjiaoxing.png "openglessanjiaoxing")

最后，因为传递数据给OpenGL的是Buffer类型，所有都要装配到ByteBuffer里面去。

#### 3.3.2 初始化着色器和纹理

把3.1的着色器代码装载，创建程序ID，然后获取属性索引：

{% highlight java %}

muMVPMatrixHandle = GLES20.glGetUniformLocation(mProgram, "uMVPMatrix");
maPositionHandle = GLES20.glGetAttribLocation(mProgram, "aPosition");
maTexCoorHandle = GLES20.glGetAttribLocation(mProgram, "aTexCoor");

{% endhighlight %}

由2.1获取得到纹理ID以后，然后把bitmap映射到纹理：

{% highlight java %}

//绑定纹理，并指定纹理的采样方式
GLES20.glBindTexture(GLES20.GL_TEXTURE_2D, mTextureId);
GLES20.glTexParameterf(GLES20.GL_TEXTURE_2D, GLES20.GL_TEXTURE_MIN_FILTER, GLES20.GL_NEAREST);
GLES20.glTexParameterf(GLES20.GL_TEXTURE_2D, GLES20.GL_TEXTURE_MAG_FILTER, GLES20.GL_LINEAR);
GLES20.glTexParameterf(GLES20.GL_TEXTURE_2D, GLES20.GL_TEXTURE_WRAP_S, GLES20.GL_CLAMP_TO_EDGE);
GLES20.glTexParameterf(GLES20.GL_TEXTURE_2D, GLES20.GL_TEXTURE_WRAP_T, GLES20.GL_CLAMP_TO_EDGE);

//纹理类型在OpenGL ES中必须为GL10.GL_TEXTURE_2D
//第二个参数是纹理的层次，0表示基本图像层，可以理解为直接贴图
//最后一个参数是纹理边框尺寸
GLUtils.texImage2D(GLES20.GL_TEXTURE_2D, 0, mBitmap, 0);

mBitmap.recycle();

{% endhighlight %}

关于纹理的采样方式，有多钟不同算法的，效率和效果各不相同，这里不解释。

#### 3.3.3 绘制

最后是计算矩阵，把各种数据传递下去管线，调用接口进行绘制，代码如下：

{% highlight java %}

public void drawDanmaku() {
    //使用3.3.2创建的shader程序ID
    GLES20.glUseProgram(mProgram);

    //初始化矩阵
    MatrixUtils.setInitStack();

    //首先把弹幕移动到右上角，由3.3.1可以理解这里
    MatrixUtils.transtate(2.0f, 0, 0);

    //弹幕平移，同理坐标放大两倍
    float unitY = -offsetY / mViewHeight * 2.0f;
    float unitX = -offsetX / mViewWidth * 2.0f;
    MatrixUtils.transtate(unitX, unitY, 0);

    //将最终变换矩阵传入shader程序
    GLES20.glUniformMatrix4fv(muMVPMatrixHandle, 1, false, MatrixUtils.getFinalMatrix(), 0);

    //传递顶点位置数据
    //坐标是xyz三维，所以size是3，每个float是4个字节，所以stride是3 * 4
    GLES20.glVertexAttribPointer(maPositionHandle, 3, GLES20.GL_FLOAT, false, 3 * 4, mVertexBuffer);

    //传递纹理坐标数据
    //坐标是xy二维的，所以size是2
    GLES20.glVertexAttribPointer(maTexCoorHandle, 2, GLES20.GL_FLOAT, false, 2 * 4, mTexCoorBuffer);

    //允许使用坐标数据数组
    GLES20.glEnableVertexAttribArray(maPositionHandle);
    GLES20.glEnableVertexAttribArray(maTexCoorHandle);

    //绑定纹理
    GLES20.glActiveTexture(GLES20.GL_TEXTURE0);
    GLES20.glBindTexture(GLES20.GL_TEXTURE_2D, mTextureId);

    //绘制纹理矩形
    GLES20.glDrawArrays(GLES20.GL_TRIANGLE_STRIP, 0, mVertexCount);
}

{% endhighlight %}

### 3.4 应用与总结

把ZGDanmakuView放到布局里面，发送弹幕的时候首先把文字绘制到bitmap里面去，然后创建一个ZGDanmaku类，最后传给Renderer去渲染绘制。

用GLSurfaceView实现，设置Renderer，初始化读取着色器代码，坐标数据，把弹幕映射到纹理，然后每帧回调中计算变换矩阵，最后把这些数据传递给管线去渲染绘制。

## 4 其他

关于片元着色器的使用，我还有一个应用，[Android OpenGLES学习之实现红蓝3D播放器]()。