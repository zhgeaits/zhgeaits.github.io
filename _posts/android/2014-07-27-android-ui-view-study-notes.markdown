---
layout: post
title:  "Android的View绘制学习"
date:   2014-07-27 10:00:03
categories: android
supertype: career
type: android
---

## 1 什么是View

最初接触View的概念的时候应该是在大学学习JavaWeb的时候了，那时候学习到[MVC](http://zhgeaits.me/javaweb/2011/04/28/javaweb-struts-study-notes.html)模式，其中的V即是View，通常翻译为视图，即界面上的东西，GUI(Graphic User Interface)方面的控件，包括图形，窗口，组件等等。而在Android，它是比这些泛化概念而具体的一个类，从这个类衍生出几乎所有的图形界面，而Window又是另外的一个类。

引用网上一句话：

>Android系统中的所有UI类都是建立在View和ViewGroup这两个类的基础上的。所有View的子类成为”Widget”，就是控件；所有ViewGroup的子类成为”Layout”，就是布局。View和ViewGroup之间采用了组合设计模式，可以使得“部分-整体”同等对待。ViewGroup作为布局容器类的最上层，布局容器里面又可以有View和ViewGroup。

View是最顶层的界面类，ViewGroup是继承View的抽象类，ViewGroup是一组view的集合。View又是所有界面的父类引用，View引用可以指向ViewGroup，实现了多态。

通常我们理解Activity为一个界面，那是不够准确的，真正的界面是View，Activity只是一个组件，它更像是一个控制器的角色，在Activity里面，我们只是调用了一个setContentView()的方法来把xml文件编写的界面使用。如下图所示，一个Activity含有一个Window，而这个Window的实例是PhoneWindow；Window包含了一个DecorView，这个DecorView才是正在看得见的界面。注意到除了Activity并不是界面。

<img src="/image/android_view.png" width="500px">

DecorView实际上是一个FrameLayout，底下嵌套了LinearLayout，上面放的是TitleBar，下面放的是ContentView，从id可以看出为什么叫setContentView。这个TitleBar是ActionBar，通常我们开发都不会使用，直接取消掉了的。DecorView不是SDK的代码，关于更多只能从android的源码去获取了。ContentView是一个FrameLayout，我们所有的View都是在这里开始的。

## 2 View的绘制流程

根据我们的经验，编写界面的xml文件是一个树状的结构，父节点通常是布局Layout，也就是一个ViewGroup，而叶子节点则是一个View，符合了ViewGroup包含View的关系。在Activity创建成功以后，变回创建ViewRoot对象ViewRootImpl，并与DecorView联系起来，这是底层系统源码，SDK上次无法查看的。View的绘制流程是从ViewRoot的peformTraversals方法开始的，分别对view进行测量，布局和绘制，最终才会显示在屏幕上。

由于是树状结构，所以从顶级的DecorView开始，分别往下遍历三个流程，实际是一个递归的过程。

### 2.1 Measure测量过程

这是从`performMeasure()`方法开始，调用到`measure()`方法，然后会调用到`onMeasure()`方法，每个View都会经历到测量，如果是ViewGroup，则会递归测量子View。最终的结果是得到每一个View/ViewGroup的宽高。

#### 2.1.1 关于MeasureSpec

在编写界面的时候，一般我们通过三种方式来确定View的宽高：match_parent、wrap_content和固定的尺寸。并且别的View，或者父节点也是由着三种方式来确定尺寸，那么将会出现很不确定的情况，如果所有的view都是使用固定的尺寸，那么根本就不需要测量的流程了。正是因为这三种模式，view的尺寸是收到父节点的影响的，所以才需要测量。

在测量的过程中，是由上至下的递归过程，需要传递mode和父节点的size这些参数，MeasureSpec就是这样的参数，它是一个32位的int类型，高两位是mode，剩下的30位是size。注意到的是，width和height分别对应一个int的MeasureSpec。

下面是三种Mode

- UNSPECIFIED，父容器不对View有任何限制，要多大给多大，一般用于系统内部。
- EXACTLY，父容器已经检测所需的精确大小，就是size的值，通常是match_parent和具体的数值两种情况。
- AT_MOST，父容器指定了一个可用的大小，就是size的值，子View不能超出这个值，通常对应于wrap_content情况。

#### 2.1.2 测量过程

从DecorView开始，它的MeasureSpec有window的尺寸和自身的LayoutParam决定，然后通过`performMeasure()`把值传递给子View，具体可用查看`getRootMeasureSpec()`方法。子View拿到父View的MeasureSpec以后，结合自身的LayoutParam，包括padding和margin等，然后计算出自己的MeasureSpec，具体可用查看`measureChildWidthMargins()`和`getChildMeasureSpec()`方法，最后子View拿到MeasureSpec以后就会调用`child.measure(childWidthMeasureSpec, childHeightMeasureSpec)`方法了，这正是上所说的重要流程。有些方法都是系统的源码，网上搜一下就有了，其他的自行去看View里面就行。所以子View的MeasureSpec收到父View的影响。

1.如果子View是一个view

在子view的`measure()`方法里面会触发调用到`onMeasure()`方法，这两个方法都可以直接在View里面查看，最终在onMeasure方法里面会调用到`setMeasuredDimension()`，参数正是测量的宽高了。注意到的是，measure()方法是是final的，我们不能重写，只能重写onMeasure()方法了，通常自定义View的时候，需要重写这个方法来计算宽高。

2.如果子View是一个ViewGroup

ViewGroup是一个抽象类，它没有重写View的onMeasure方法，提供了measureChildren的方法，具体onMeasure的实现交给了子类，例如LinearLayout等等，因为它是需要递归去测量子View的尺寸的。

#### 2.1.3 测量结果获取的时机

注意到的是测量过程和Activity的生命周期是不同步的，在生命周期的方法里面获取测量结果是不安全的，应该在一下三个时机获取：

- onWindowFocusChanged 这个方法，在Activity那个blog post有介绍了。
- view.post(runnable) 调用view的post方法来获取
- 使用ViewTreeObserver的回调

另外较好的是在onLayout方法里面获取测量宽高。

### 2.2 Layout布局过程

这是从`performLayout()`方法开始，调用到'layout()'方法，然后会调用到`onLayout()`的方法，同样道理递归下去，最终结果是得到View的四个顶点坐标，left，right，top，bottom和实际View的宽高。

在根View开始的时候如下所示：

{% highlight java %}

private void performLayout() {
    ......
    mView.layout(0, 0, mView.getMeasuredWidth(), mView.getMeasuredHeight());
    ......
}

{% endhighlight %}

View的layout的方法如下：

{% highlight java %}

public void layout(int l, int t, int r, int b) {
    if ((mPrivateFlags3 & PFLAG3_MEASURE_NEEDED_BEFORE_LAYOUT) != 0) {
        onMeasure(mOldWidthMeasureSpec, mOldHeightMeasureSpec);
        mPrivateFlags3 &= ~PFLAG3_MEASURE_NEEDED_BEFORE_LAYOUT;
    }

    int oldL = mLeft;
    int oldT = mTop;
    int oldB = mBottom;
    int oldR = mRight;

    boolean changed = isLayoutModeOptical(mParent) ?
            setOpticalFrame(l, t, r, b) : setFrame(l, t, r, b);

    if (changed || (mPrivateFlags & PFLAG_LAYOUT_REQUIRED) == PFLAG_LAYOUT_REQUIRED) {
        onLayout(changed, l, t, r, b);
        mPrivateFlags &= ~PFLAG_LAYOUT_REQUIRED;

        ListenerInfo li = mListenerInfo;
        if (li != null && li.mOnLayoutChangeListeners != null) {
            ArrayList<OnLayoutChangeListener> listenersCopy =
                    (ArrayList<OnLayoutChangeListener>)li.mOnLayoutChangeListeners.clone();
            int numListeners = listenersCopy.size();
            for (int i = 0; i < numListeners; ++i) {
                listenersCopy.get(i).onLayoutChange(this, l, t, r, b, oldL, oldT, oldR, oldB);
            }
        }
    }

    mPrivateFlags &= ~PFLAG_FORCE_LAYOUT;
    mPrivateFlags3 |= PFLAG3_IS_LAID_OUT;
}

{% endhighlight %}

可以看到layout是确定View本身的位置，初始的时候即是DecorView的位置，显然，DecorView的位置是DecorView的尺寸，然后里面会调用到onLayout，但是View和ViewGroup的onLayout方法都没有实现，交给了子类具体去实现。可以去看LinearLayout的onLayout方法。

要理解到的是，确定view的位置只需要left、top和测量宽高即可。

onLayout方法则会确所有子元素的位置。如果子元素是View则会用到View的测量宽高调用layout方法来确定位置；如果子元素是ViewGroup，则确定ViewGroup位置后会递归调用layout方法往下调用直到调用完毕。

由上可知，测量宽高形成于measure过程，最终宽高形成与layout过程，两者是相等的，只是赋值时机不同。

### 2.3 Draw绘制过程

这是从`performDraw()`方法开始，调用到'draw()'方法，然后会调用到`onDraw()`方法，唯一不同的是，这时候是通过`dispatchDraw()`方法了来传递到子View去绘制。

在根View执行performDraw()方法如下：

{% highlight java %}

private void performTraversals() {
    ......
    final Rect dirty = mDirty;
    ......
    canvas = mSurface.lockCanvas(dirty);
    ......
    mView.draw(canvas);
    ......
}

{% endhighlight %}

可以直接去看View的draw方法，里面的注释非常明晰，分了6个步骤，主要是绘制背景`backgroud.draw(canvas)`，绘制自己`onDraw()`，绘制子元素`dispatchDraw()`，绘制装饰`onDrawScrollBars()`。

其中View是没有实现dispatchDraw()的，但是ViewGroup有实现，它是会递归去绘制所有的子View的。一般我们自定义View，需要实现onDraw()方法即可。

## 3 View的事件体系



## 4 自定义View控件

由上面知道了自定义控件需要处理view的测量、布局和绘制过程，还需要处理事件冲突等等的问题。通常自定义的View有以下几种情况：

### 4.1 继承View

这是比较原生的做法，需要做的事情比较多，但是一般我们是重写onDraw()方法，如果要支持wrap_content和padding属性，这是需要我们自己处理的。如果是wrap_content模式，对应specMode是AT_MOST，采用的specSize是parentSize，结果和match_parent一样，所以要处理的话就在重写onMeasure方法，里面使用一个默认的值，对于TextView这些，它是另外计算字体的宽高的。

这种情况完全是不用重写onLayout方法的，一般是onMeasure和onDraw方法即可。

### 4.2 继承ViewGroup

这也是比较原生的做法，实现不同的布局，这时候我们是必须重写测量、布局的过程，那是比较复杂的事情。

### 4.3 继承特定的View

一般是扩展某些View的功能。

### 4.4 继承特定的ViewGroup

一般是扩展某些布局的功能。

### 4.5 自定义控件属性

当自定义一个控件后，可以给这个控件自定义更多属性，例如我自定义一个ImageView，给它一个属性，是否圆角。

**1.** 在res/values下创建attrs.xml文件，里面定义好属性名称和类型:  

{% highlight xml %}
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <declare-styleable name="roundedimageview">
        <attr name="border_thickness" format="dimension" />
        <attr name="border_inside_color" format="color" />
        <attr name="border_outside_color" format="color"></attr>
    </declare-styleable>
</resources>
{% endhighlight %}

name="roundedimageview" 是属性的总体名称  
name="border_thickness" 是具体的每一个属性  
format="color" 是说明这个属性只能填颜色值  

**2.** 编写自定义控件类

**3.** 在布局文件里面引用新的命名空间。

{% highlight xml %}
xmlns:zg="http://schemas.android.com/apk/res/org.zhangge.almightyzgbox_android" 
{% endhighlight %}

org.zhangge.almightyzgbox_android这个是我的包名

使用新的自定义控件和自定义属性

{% highlight xml %}
<org.zhangge.ui.image.ZGImageView  
            android:id="@+id/myView"  
            android:layout_width="fill_parent"  
            android:layout_height="wrap_content"  
            android:layout_marginBottom="5dip"  
            zg:border_thickness="XXX" 
            zg:border_inside_color="XXX"  
            zg:border_outside_color="XXX" />
{% endhighlight %}

**4.** 在自定义控件类里面的构造方法获取这个属性值：

{% highlight java %}
//获得实例  
TypedArray typeArray = context.obtainStyledAttributes(attrs,  
		R.styleable.FlowIndicator);  
//从typeArray获取相应值，第二个参数为默认值，如第一个参数在atts.xml中没有定义，返回第二个参数值  

thickness = typeArray.getDimension(R.styleable.roundedimageview_border_thickness, 9);  
point_normal_color = typeArray.getColor(  
		R.styleable.roundedimageview_border_inside_color, 0x000000);  
point_seleted_color = typeArray.getColor(  
		R.styleable.roundedimageview_border_outside_color, 0xffff07); 
		
typeArray.recycle();
{% endhighlight %}

获得了属性，就可以自己去构造这个view了。注意：

属性的命名规则是：采取的名字_属性这种格式。例如：R.styleable.roundedimageview_border_thickness

还有：一定要执行typeArray.recycle()回收。
