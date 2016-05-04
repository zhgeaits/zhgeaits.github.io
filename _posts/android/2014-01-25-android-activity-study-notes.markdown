---
layout: post
title:  "Android的四大组件之Activity"
date:   2014-01-25 15:00:01
categories: android
supertype: career
type: android
---

## 1 什么是Activity

刚学习Android的时候，会不理解这是什么，因为直译是“活动”，是实际上用起来的时候却是一个界面，那为什么会叫这名字呢？直接叫window之类的名字不是更好么？因为它不仅仅是界面！而真正界面也不是它，只是调用了`setContentView()`而已。

实际上，View是真正的界面内容，在xml文件中编写；Window是一个显示屏，显示view的；activity是一个控制器，调用window来显示view，setContentView的实质是：

>getWindow().setContentView(LayoutInflater.from(this).inflate(R.layout.main, null))；

如果学过iOS会知道，这跟OC里面的Controller很像，没错，Activity充当的就是MVC模式里面的控制器角色。但是，随着技术的发展，MVC模式难以要解决越来越复杂的业务，开发者几乎什么逻辑代码都在Activity里面编写，会变得越来越臃肿，于是就出现了分层的架构，MVP，MVVC模式等等。

## 2 生命周期

虽然是一个界面组件，但是最重要的是它有生命周期，是活的，而界面是静态的，顶层已经包含了window这些类。因此只有理解这个才能写得好程序。
  
如下图是官网给出的Activity的整个生命周期：
  
![alt lifecycle](/image/activity_lifecycle.gif "lifecycle")  

可以看出生命周期回调的方法都是配对出现的，`onCreate()`和`onDestroy()`，`onStart()`和`onStop()`，`onResume()`和`onPause()`。但是也有一个`onRestart()`是单独的。

- `onCreate()`和`onDestroy()`都是会只执行一次，创建或者销毁了，我们大部分代码都是在创建的时机去编写的，初始化界面，例如获取view，和初始化数据等等，但是也要注意不要在这里做太多操作，例如耗时的一些操作，不然会影响打开界面的速度。最后被销毁的时候，onDestroy里面做一些资源回收和释放的操作。

- `onStart()`和`onStop()`这一对我们都用得比较少，当界面创建完毕以后就会调用onStart()，表示界面已经可见了，但是还没有位于前台（foreground），即还是不能交互。当界面不可以见的时候就会调用onStop()了，最常发生的就是用户按了home键切换到主界面或者启动新的一个activity；这里不要做太多耗时的操作，不然导致应用ANR。

- `onResume()`和`onPause()`这是我们用得最多的地方了，到onResume的时候表示activity已经是可见了，并处于前台开始活动了；当activity切换出前台了，就会调用onPause，activity这个时候还是可见的。这里是不应该做耗时的操作的。

- `onRestart()`是在onStop以后重现显示界面的时候调用，相对比较少在这里用上。

>对于前台（foreground），我的理解是可见并可以交互，即获得了焦点，对于activity来说，前台应该只有一个activity。例如activity之上出现了一个对话框的时候，activity就会从前台切换到了后台了，虽然是可见，但是不可交互了。对话框消失，activity重新切换到前台，就会调用onResume了。

如果activity A启动activity B，那么是会先执行A的onPause，再执行B的onResume的，最后才会执行A的onStop。

### 2.1 关于`onStart()`和`onStop()`，`onResume()`和`onPause()`的区别

两对看似相同的作用，实际上有很多的不同，根据上面的分析，`onStart()`和`onStop()`是从可见的角度来回调，而`onResume()`和`onPause()`是从是否位于前台的角度来回调的。

### 2.2 真正的可见时机

实际上，onStart, onResume都不是真正visible的时间点，真正的visible时间点是onWindowFocusChanged()函数被执行时。  

> public void onWindowFocusChanged(boolean hasFocus)

从onWindowFocusChanged被执行起，用户可以与应用进行交互了，而这之前，对用户的操作需要做一点限制。实际上这个方法的含义是，View已经初始化完毕了，宽高已经准确计算好了，一般是在这里来获取view的宽高。而且，这个方法是会被调用多次的。

### 2.3 关于`onSaveInstanceState()`和`onRestoreInstanceState()`

这两个方法只有在activity异常被终止的情况下才会被调用，例如内存不足会杀死一些优先级低的activity，另外，屏幕旋转也是算异常终止重建的。我们需要在onSaveInstanceState的时候把数据保存在Bundle里面，然后当activity重建的时候，在onCreate方法里面拿到Bundle判断是否为null来恢复数据，之后会调用onRestoreInstanceState，同样可以在这里恢复数据，这个方法之后才会调用onStart()。

## 3 启动模式LaunchMode

在AndroidManifest.xml里面配置一个Activity的时候，有这么一个属性`android:launchMode`，它的值有四种，分别是standard、singleTop、singleTask和singleInstance。要理解这些值就必须先理解任务栈：

试想一下，用户打开了一系列的界面，系统该用什么数据结构来管理这些界面来为用户导航？当用户按下返回键的时候，界面是需要一个个的消失和弹出显示。显然，栈是最适合的了。新建一个就压栈，返回就出栈。启动应用的时候会默认创建一个栈，名字是包名，默认的情况，我们都是在使用这个栈的，因为如果startActivity，新起来的activity会加入到启动它的那个activity所属的栈。当然，我们也是可以新建不同的栈的。

- standard模式：这是标准的模式，也是默认的值，每次startActivity都会新创建activity，然后进入启动它的activity所属的栈。但是，如果我们用ApplicationContext来启动就可能会报错了，因为它没有栈，需要我们新建一个。在Intent里面添加一个FLAG_ACTIVITY_NEW_TASK标记即可。

- singleTop模式：这是栈顶复用模式，意思是只有activity位于栈顶的时候就会复用，其他情况都会新创建activity。然后会触发调用`onNewIntent()`方法。

- singleTask模式：这是栈内复用模式，意思是只要栈里面有这个activity就会复用了，不管是不是在栈顶，如果不是在栈顶，它就会把上面的activity都出栈销毁。通常它会配合TaskAffinity参数使用，它是用来指定用哪个栈的，默认的值是应用的包名。如果没有配置taskAffinity的话，应该是会自动生成一个名字。

>配置的时候这样使用`android:taskAffinity=""`，指定一个名字即可。

- singleInstance：这是单任务单实例模式，启动的时候回创建一个新的任务栈，只存放这一个activity，以后都会复用，除非销毁了。

### 3.1 标记位

在startActivity的时候，可以调用这个方法`intent.addFlags()`，它也是可以用来设置启动模式的，但是不完全和上面四种重合，它有很多种标记，而且优先级比文件配置的高；常用的标记是下面几种：

- FLAG_ACTIVITY_NEW_TASK: 和singleTask一样

- FLAG_ACTIVITY_SINGLE_TOP：和singleTop一样

- FLAG_ACTIVITY_CLEAR_TOP：会清除栈上面的activity。

## 4 IntentFilter配对规则

一般我们启动Activity的时候都是显式指定启动的，很少用到隐式启动的，但是启动应用的MainActivity肯定是隐式的，因为我们配置了：

{% highlight xml %}

<intent-filter>
    <action android:name="android.intent.action.MAIN" />
    <category android:name="android.intent.category.DEFAULT" />
    <category android:name="android.intent.category.LAUNCHER" />
</intent-filter>

{% endhighlight %}

所以系统能够默认启动MainActivity。对于其他的Activity，我们也是可以配置intent-filter来让它支持隐式启动的。我们可以为activity配置多个intent-filter，而intent-filter里面可以配置多个action, category和data标签。系统只要匹配其中一个intent-filter成功就可以了，但是需要同时匹配intent-filter底下的action，category和data。

- action标签：intent只能够set一个action，只要这个action匹配了其中一个就算成功了。如果没有设置action那是会匹配失败的。

- category标签：intent可以add多个category，只有这些category都匹配才能算成功，如果Intent没有add category，系统是会默认添加android.intent.category.DEFAULT的，所以如果我们一般都配置这个，基本上都能匹配成功。

- data标签：这个比较复杂，包含了mimeType和URI，mimeType是媒体类型，比较熟悉了，URI包含比较多信息，如下：

`<scheme>://<host>:<port>/[<path>|<pathPrefix>|<pathPattern>]`

在intent里面调用`setDataAndType()`来设置的。另外，通过在MainActivity配置data，我们可以让别的应用来打开，例如我们的app是播放器，然后配置这个来支持打开视频文件。或者配置使得可以用js打开应用。一个例子如下：

{% highlight xml %}

<data
	android:mimeType="video/*"
    android:scheme="ddt-share-footer-btn"
    android:host="ddt.ddt"
    android:pathPrefix="/ddt">
</data>

{% endhighlight %}

data的配置规则和action是一样的，如果设置了就intent里面必须有data匹配成功。

## 5 跳转

当一个Activity有很多个入口和出口的时候，就是它可能由不同多个Activity启动，而且它也可以去到很多不同的Activity的时候，Activity就需要处理多种情况来改变状态了。

Android给我们提供了requestCode和resultCode的方式处理，例如，从A去到B，那么启动Activity的时候传入requestCode：

>context.startActivityForResult(intent, requestCode);

然后当B要关闭的时候，把resultCode传回去：

>setResult(resultCode);  
>finish();

最后在A要重写这个方法，来处理requestCode和resultCode：

{% highlight java %}  
@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
    super.onActivityResult(requestCode, resultCode, data);
    if(requestCode == a) {
        if (resultCode == b) {
            //do something
        }
    }
}  
{% endhighlight %}

这个看起来很简单，当我在一个比较复杂的业务中使用起来的时候，还是需要认真理解一下的，不然的话，你会发现，你定义的requestCode和resultCode很乱。

其实，这两个code只是一个消息而已，A进入B，会带着requestCode，B不处理requestCode，因为不会影响B，只有intent里面的数据才会决定B，B才不会给不同的requestCode传不同的resultCode，因而B只管自己结束时有哪些状态罢了，**所以resultCode定义在B**，并把resultCode传回去，同时原封不动地隐含传requestCode回去；**因此requestCode是定义在A的**，A需要处理是B还是D回来的resultCode来改变状态。这就是状态机啊。核心是A的状态改变，依赖于出口动作所带回来的消息而改变状态，即到B还是到D回来的resultCode。

## Window

Window是一个抽象类，官网上说他的唯一的实现是PhoneWindow，  
官网介绍window：It provides standard UI policies such as a background, title area, default key processing, etc.  
window都有一个View，称之为DecorView，是主窗口中的顶级view，实际上就是ViewGroup。  
并不是一个app只有一个window，而是当有东西需要显示在界面上，就需要一个window，就是说一个界面，包括内容，逻辑和窗口，三者同时需要。启动activity需要window，启动dialog需要window，
只要启动的控件有getWindow()方法都需要一个window来显示。那么这些window的管理，增删回收等就需要一个Manager了。  
可以修改设置window的参数，也可以设置window里面的view的参数，  没有认真学习过这些类，是完全不能理解这些参数设置的意义的。  
设置全屏：  
getWindow().setFlags(WindowManager.LayoutParams.FLAG_FULLSCREEN, WindowManager.LayoutParams.FLAG_FULLSCREEN);

WindowManager主要用来管理窗口的一些状态、属性、view增加、删除、更新、窗口顺序、消息收集和处理等。在WindowManager中还有一个重要的静态类LayoutParams.通过它可以设置和获得当前窗口的一些属性。

在自定义一个弹窗的时候，继承Dialog，然后获取window，设置他的大小，位置，例如在底部，然后再设置动画，则可以实现底部弹窗的效果。但是popwindow也挺好用的。


## starting window

在做应用的时候，很多初始化的东西都放在了application里面去，例如初始化一个sdk。然后导致启动第一个activity的时候很慢，会卡在一个黑屏或者白屏的地方。这是因为应用还没有在运行，系统会为这个Activity所属的应用创建一个进程，但进程的创建与初始化都需要时间，于是系统就会先创建一个starting window，或者叫preview window。Starting Window就是一个用于在应用程序进程创建并初始化成功前显示的临时窗口，拥有的Window Type是TYPE_APPLICATION_STARTING。

启动慢这个问题无法解决，因为你的进程就是需要启动那么久，我们只能修改这个starting window的主题来改进体验，就是在第一个activity，如splash activity那里给他单独设置一个theme，为他设置android:windowBackground,android:windowIsTranslucent属性，为透明，或者一张图片也好。

