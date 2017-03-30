---
layout: post
title:  "Understanding Android's LayoutInflater"
date:   2017-03-30 11:00:03
categories: android
supertype: career
type: android
---

## 1. What's LayoutInflater

什么是LayoutInflater，在学习android的时候，一般在ListView或者GridView的时候，在Adapter的getView()方法里面会用到这个，其作用就是为ListView的每一个item生成一个view，一般来说我们需要另外写一个布局传给LayoutInflater即可，用法如下：

{% highlight java %}
LayoutInflater.from(context).inflate(R.layout.activity_main_list_item, null);
{% endhighlight %}

它的作用和功能都很明显了，再加上`Inflater`这个单词的意思是充气机，它把xml布局比喻为憋的气球或者是轮胎等等，然后给它充气就成了一个可视的view对象了。对比一下HTML，在网页上来说也只是一个骨架而已，还需要css和js来装饰和增加动效等等，而转化为一个可视的网页的时候，我们用到的单词是`Render`，意思是渲染。无独有偶，其实原理都是一样的，我们理解即可。但是依然是有区别的，具体是什么，最后就知道了。

>关于Android View的学习，曾经写过一篇[Blog](http://zhgeaits.me/android/2014/07/27/android-ui-view-study-notes.html)记录，当时关于LayoutInflater这块是没有深入学习，只是在理解到Activity里面setContentView的时候本质是调用了LayoutInflater来生成Activity的界面。而更多的是学习了View的绘制原理、事件，以及自定义View的相关知识，于是现在有空再来学习一下LayoutInflater。

## 2. How to new LayoutInflater

使用LayoutInflater通常有以下几种方法获取LayoutInflater的对象。

**第一种方法：**直接通过context获取一个系统服务的接口，传入Context.LAYOUT_INFLATER_SERVICE即可获取一个LayoutInflater的对象，这里我们就会产生疑惑，这是一个系统服务吗？

{% highlight java %}
LayoutInflater inflater = (LayoutInflater) context.getSystemService(Context.LAYOUT_INFLATER_SERVICE);
{% endhighlight %}

我们再看看这个常量的定义如下：

{% highlight java %}
/**
 * Use with {@link #getSystemService} to retrieve a
 * {@link android.view.LayoutInflater} for inflating layout resources in this
 * context.
 *
 * @see #getSystemService
 * @see android.view.LayoutInflater
 */
public static final String LAYOUT_INFLATER_SERVICE = "layout_inflater";
{% endhighlight %}

似乎并没有更多的注释介绍，而是简单说明了一下LayoutInflater，然后建议我们自己去看这个类。

**第二种方法：**这也是我们最常用的方法，也是设计模式里面推荐的方法，直接通过LayoutInflater的静态方法获取实例，只需要传入一个Context对象即可。这里我们又可以产生一个疑问，LayoutInflater和Context是什么样的关系？关于Context的本质以后需要学习再起Blog记录。

{% highlight java %}
LayoutInflater inflater = LayoutInflater.from(context);
{% endhighlight %}

再深入看看这个`from()`方法的内容，如下：

{% highlight java %}
/**
 * Obtains the LayoutInflater from the given context.
 */
public static LayoutInflater from(Context context) {
    LayoutInflater LayoutInflater =
            (LayoutInflater) context.getSystemService(Context.LAYOUT_INFLATER_SERVICE);
    if (LayoutInflater == null) {
        throw new AssertionError("LayoutInflater not found.");
    }
    return LayoutInflater;
}
{% endhighlight %}

发现原来依然调用了方法一的接口。

**第三种方法：**在Activity里面可以直接调用`getLayoutInflater()`方法来获取：

{% highlight java %}
LayoutInflater inflater = Activity.getLayoutInflater();
{% endhighlight %}

再深入看这个方法的内容，如下：

{% highlight java %}

/**
 * Convenience for calling
 * {@link android.view.Window#getLayoutInflater}.
 */
@NonNull
public LayoutInflater getLayoutInflater() {
    return getWindow().getLayoutInflater();
}
{% endhighlight %}

可以看到是调用了Window对象的方法来获取Inflater，在SDK里面只能看到Window这个抽象类，看不到它的实现类PhoneWindow，通过GitHub上面Android的源码即可看到[PhoneWindow](https://github.com/android/platform_frameworks_base/blob/d59921149bb5948ffbcb9a9e832e9ac1538e05a0/core/java/com/android/internal/policy/PhoneWindow.java)的源码，其中在构造方法如此：

{% highlight java %}
public PhoneWindow(Context context) {
    super(context);
    mLayoutInflater = LayoutInflater.from(context);
}
{% endhighlight %}

由此可知是调用了方法二来获取对象的。

**第四种方法：**有时候看到别人的代码是直接用View的静态方法来调用inflate方法来渲染类，差点以为是View也有了inflate的功能，点进去一看，实际上是调用了LayoutInflater的方法。

{% highlight java %}
View.inflate(context, R.layout.view_layout, null);

/**
 * Inflate a view from an XML resource.  This convenience method wraps the {@link
 * LayoutInflater} class, which provides a full range of options for view inflation.
 *
 * @param context The Context object for your activity or application.
 * @param resource The resource ID to inflate
 * @param root A view group that will be the parent.  Used to properly inflate the
 * layout_* parameters.
 * @see LayoutInflater
 */
public static View inflate(Context context, int resource, ViewGroup root) {
    LayoutInflater factory = LayoutInflater.from(context);
    return factory.inflate(resource, root);
}
{% endhighlight %}

**总结上面的几种方法，实际上都是调用了Context.getSystemService()方法，那么我们再深入看看是如何获取这个系统服务的：**

{% highlight java %}
public Object getSystemService(String name) {
    if (LAYOUT_INFLATER_SERVICE.equals(name)) {
        if (mInflater == null) {
            mInflater = LayoutInflater.from(getBaseContext()).cloneInContext(this);
        }
        return mInflater;
    }
    return getBaseContext().getSystemService(name);
}
{% endhighlight %}

这就神奇了，原来LayoutInflater并不是一个系统服务，它区分了其他的系统服务，单独调用了`LayoutInflater.from(getBaseContext()).cloneInContext(this);`，那么问题来了，又调回了LayoutInflater的静态方法from()，这难道不是死循环的递归了吗？

非也，关键在于Context这个对象并不相同。假设在`Activity A`里面调用了`LayoutInflater.from(this)`来获取一个对象，那么底下调用了这个Activity A的`getSystemService()`方法，首次的时候mInflater为null，于是再次调用`LayoutInflater.from(getBaseContext())`，注意到的是传入的并不是this了，如果还是this，那么就肯定会死循环递归了。看看`getBaseContext()`的描述如下：

{% highlight java %}
/**
 * @return the base context as set by the constructor or setBaseContext
 */
public Context getBaseContext() {
    return mBase;
}
{% endhighlight %}

它是ContextWrapper的属性，上面说明的是通过构造方法或者setter方法设置过来的：

{% highlight java %}

public ContextWrapper(Context base) {
    mBase = base;
}
{% endhighlight %}

关于Context的这个机制，似乎目前并不了解，但是不影响我们理解，而且我们也可以大胆猜想，这可能是Application的Context。由于Activity是系统组件，由系统实例化的，调用构造方法的时候传入的Base Context都是系统的事情，而且，Base Context里面的mInflater一定不为null！应该是应用里面唯一的。

获取得到Base Context的Inflater以后，调用了LayoutInflater的`cloneInContext()`方法，而且传入的是this参数了，看看这个方法的描述:

{% highlight java %}
/**
 * Create a copy of the existing LayoutInflater object, with the copy
 * pointing to a different Context than the original.  This is used by
 * {@link ContextThemeWrapper} to create a new LayoutInflater to go along
 * with the new Context theme.
 * 
 * @param newContext The new Context to associate with the new LayoutInflater.
 * May be the same as the original Context if desired.
 * 
 * @return Returns a brand spanking new LayoutInflater object associated with
 * the given Context.
 */
public abstract LayoutInflater cloneInContext(Context newContext);
{% endhighlight %}

原来LayoutInflater也是抽象类，跟Window一样，应该也是有实现类我们看不到。上面的描述很简单，意思就是使用克隆一个新的Inflater指向了新的Context，之前使用来创建新的Context主题。

## 3. About LayoutInflater's inflate() Method

我们拿到了LayoutInflater对象以后，那么这个类到底是什么，先看看注释：

{% highlight java %}
/**
 * Instantiates a layout XML file into its corresponding {@link android.view.View}
 * objects. It is never used directly. Instead, use
 * {@link android.app.Activity#getLayoutInflater()} or
 * {@link Context#getSystemService} to retrieve a standard LayoutInflater instance
 * that is already hooked up to the current context and correctly configured
 * for the device you are running on.  For example:
 *
 * <pre>LayoutInflater inflater = (LayoutInflater)context.getSystemService
 *      (Context.LAYOUT_INFLATER_SERVICE);</pre>
 * 
 * <p>
 * To create a new LayoutInflater with an additional {@link Factory} for your
 * own views, you can use {@link #cloneInContext} to clone an existing
 * ViewFactory, and then call {@link #setFactory} on it to include your
 * Factory.
 * 
 * <p>
 * For performance reasons, view inflation relies heavily on pre-processing of
 * XML files that is done at build time. Therefore, it is not currently possible
 * to use LayoutInflater with an XmlPullParser over a plain XML file at runtime;
 * it only works with an XmlPullParser returned from a compiled resource
 * (R.<em>something</em> file.)
 * 
 * @see Context#getSystemService
 */
{% endhighlight %}

第一句意思就是说把一个xml文件实例化成为相应的一个View对象，接着介绍了几个获取LayoutInflater对象的方法。它还介绍了cloneInContext()方法和Factory的相关，这个后面再学，这里先不管。最后说了一个性能原因，view的渲染是非常重的依赖于构建时期的预处理xml文件。因此，当前不太可能的是在运行期使用LayoutInflater和XmlPullParser来渲染一个普通简单的xml文件，只有经过编译的xml文件，XmlPullParser才能处理。

我们主要调用LayoutInflater的inflate()方法来渲染一个View，而这个方法，有两个重载的方法：

{% highlight java %}

/**
 * Inflate a new view hierarchy from the specified xml resource. Throws
 * {@link InflateException} if there is an error.
 * 
 * @param resource ID for an XML layout resource to load (e.g.,
 *        <code>R.layout.main_page</code>)
 * @param root Optional view to be the parent of the generated hierarchy.
 * @return The root View of the inflated hierarchy. If root was supplied,
 *         this is the root View; otherwise it is the root of the inflated
 *         XML file.
 */
public View inflate(int resource, ViewGroup root) {
    return inflate(resource, root, root != null);
}

/**
 * Inflate a new view hierarchy from the specified xml resource. Throws
 * {@link InflateException} if there is an error.
 * 
 * @param resource ID for an XML layout resource to load (e.g.,
 *        <code>R.layout.main_page</code>)
 * @param root Optional view to be the parent of the generated hierarchy (if
 *        <em>attachToRoot</em> is true), or else simply an object that
 *        provides a set of LayoutParams values for root of the returned
 *        hierarchy (if <em>attachToRoot</em> is false.)
 * @param attachToRoot Whether the inflated hierarchy should be attached to
 *        the root parameter? If false, root is only used to create the
 *        correct subclass of LayoutParams for the root view in the XML.
 * @return The root View of the inflated hierarchy. If root was supplied and
 *         attachToRoot is true, this is root; otherwise it is the root of
 *         the inflated XML file.
 */
public View inflate(int resource, ViewGroup root, boolean attachToRoot) {
    final Resources res = getContext().getResources();
    if (DEBUG) {
        Log.d(TAG, "INFLATING from resource: \"" + res.getResourceName(resource) + "\" ("
                + Integer.toHexString(resource) + ")");
    }

    final XmlResourceParser parser = res.getLayout(resource);
    try {
        return inflate(parser, root, attachToRoot);
    } finally {
        parser.close();
    }
}
{% endhighlight %}

可以看到第一个参数是resource ID，指向了xml布局，第二个参数root是一个ViewGroup，第三个参数attachToRoot。

看看第一个方法的注释，它实际是调用了第二个方法，它的说明第二个参数是可选的，即可以为null，非空的时候是生成的view的parent。它说返回的生成的view结构，如果第二个参数root不为空，则返回的是root，否则就是渲染生成的xml文件的root。

再看第二个方法，第二个参数root的注释增加了一些说明，如果第三个参数attachToRoot为true，则root才是生成的view的parent；如果为false，那么root只是一个简单的object对象，为生成的view的root提供了LayoutParams的参数值。因此第三个参数的注释也说明了是attach to the root parameter。再看返回的说明，如果root不为null，并且attachToRoot是true的才返回的是root。

可以看到，实际上拿到布局xml的资源id以后生成了一个`XmlResourceParser`，再次调用了`inflate()`的接口。看看这两个方法的说明：

{% highlight java %}
public View inflate(XmlPullParser parser, ViewGroup root) {
    return inflate(parser, root, root != null);
}

public View inflate(XmlPullParser parser, ViewGroup root, boolean attachToRoot) {
//...
}
{% endhighlight %}

第二个方法较长，暂时省略了。这两个方法的注释中，主要的一个是关于parser的注释，说的是解析xml的dom树，另外两个参数都是同样的注释，意思和上面是一样的。

## 4. More About inflate() Method

现在看看核心的inflate()的逻辑了，方法比较长，于是删除了一些debug的打印：

{% highlight java %}
public View inflate(XmlPullParser parser, ViewGroup root, boolean attachToRoot) {
    synchronized (mConstructorArgs) {
        final AttributeSet attrs = Xml.asAttributeSet(parser);
        Context lastContext = (Context)mConstructorArgs[0];
        mConstructorArgs[0] = mContext;
        View result = root;
        try {
            // Look for the root node.
            int type;
            while ((type = parser.next()) != XmlPullParser.START_TAG &&
                    type != XmlPullParser.END_DOCUMENT) {
                // Empty
            }
            if (type != XmlPullParser.START_TAG) {
                throw new InflateException(parser.getPositionDescription()
                        + ": No start tag found!");
            }
            final String name = parser.getName();
            if (TAG_MERGE.equals(name)) {
                if (root == null || !attachToRoot) {
                    throw new InflateException("<merge /> can be used only with a valid "
                            + "ViewGroup root and attachToRoot=true");
                }
                rInflate(parser, root, attrs, false, false);
            } else {
                // Temp is the root view that was found in the xml
                final View temp = createViewFromTag(root, name, attrs, false);
                ViewGroup.LayoutParams params = null;
                if (root != null) {
                    // Create layout params that match root, if supplied
                    params = root.generateLayoutParams(attrs);
                    if (!attachToRoot) {
                        // Set the layout params for temp if we are not
                        // attaching. (If we are, we use addView, below)
                        temp.setLayoutParams(params);
                    }
                }
                // Inflate all children under temp
                rInflate(parser, temp, attrs, true, true);
                // We are supposed to attach all the views we found (int temp)
                // to root. Do that now.
                if (root != null && attachToRoot) {
                    root.addView(temp, params);
                }
                // Decide whether to return the root that was passed in or the
                // top view found in xml.
                if (root == null || !attachToRoot) {
                    result = temp;
                }
            }
        } catch (XmlPullParserException e) {
            InflateException ex = new InflateException(e.getMessage());
            ex.initCause(e);
            throw ex;
        } catch (IOException e) {
            InflateException ex = new InflateException(
                    parser.getPositionDescription()
                    + ": " + e.getMessage());
            ex.initCause(e);
            throw ex;
        } finally {
            // Don't retain static reference on context.
            mConstructorArgs[0] = lastContext;
            mConstructorArgs[1] = null;
        }
        return result;
    }
}
{% endhighlight %}

首先是获取[AttributeSet](http://zhgeaits.me/android/2014/07/27/android-ui-view-study-notes.html)，然后把root赋值给result，它是返回值。接下来是一个while循环，上面带了注释说寻找root node，可以看到的是寻找一个type为`XmlPullParser.START_TAG`的节点，找不到就抛出异常。如果找到的是Merge节点，则调用了一个`rInflate()`并结束。

否则调用了`createViewFromTag()`来创建一个temp view，注释说明了temp是xml里面的root view。接着开始判断root是否为空的情况，如果非空的话，就调用了root的`generateLayoutParams()`来创建布局的参数，然后判断attachToRoot为false则把这个参数设置在temp。接着也是调用了`rInflate()`来渲染temp下的所有子节点。子节点渲染完毕以后判断root非空并且attachToRoot为true，则把temp这个view add到了root里面去。最后当root为空，或者attachToRoot为false的情况返回的是temp，否则返回的是root。也就是说，只有root非空，且attachToRoot为true的情况下才返回root，并把布局add到root里面去。否则只是读取了root的布局参数，返回的是xml的view。这也是符合了注释的。

## 5.About XmlPullParser、rInflate() and createViewFromTag()

点开XmlPullParser，会发现是一个interface，里面有很详细的注释，说明的就是如何解释经过与处理编译的xml文件，这点我暂时不深入去学习了。而rInflate()则是通过递归去解释每一个子节点，其中处理了`requestFocus`，`tag`，`include`，`merge`节点，而实际上都是通过调用`createViewFromTag()`生成view的。

`createViewFromTag()`实际上是根据给定的AttributeSet，然后调用了Factory类来创建view的。看到`createViewFromTag()`方法如下：

{% highlight java %}
/**
 * Creates a view from a tag name using the supplied attribute set.
 * <p>
 * If {@code inheritContext} is true and the parent is non-null, the view
 * will be inflated in parent view's context. If the view specifies a
 * &lt;theme&gt; attribute, the inflation context will be wrapped with the
 * specified theme.
 * <p>
 * Note: Default visibility so the BridgeInflater can override it.
 */
View createViewFromTag(View parent, String name, AttributeSet attrs, boolean inheritContext) {
//..
}
{% endhighlight %}

注释说明如果inheritContext为true，并且parent不为null，则使用parent view的context。如果view使用了一个主题属性，那么context也会包含这个主题。关于name是什么，看到调用createViewFromTag传过来的都是`parser.getName()`，这个是XmlPullParser的方法，查看如下：

{% highlight java %}
/**
 * For START_TAG or END_TAG events, the (local) name of the current
 * element is returned when namespaces are enabled. When namespace
 * processing is disabled, the raw name is returned.
 * For ENTITY_REF events, the entity name is returned.
 * If the current event is not START_TAG, END_TAG, or ENTITY_REF,
 * null is returned.
 * <p><b>Please note:</b> To reconstruct the raw element name
 *  when namespaces are enabled and the prefix is not null,
 * you will need to  add the prefix and a colon to localName..
 *
 */
String getName();

{% endhighlight %}

由此可以知道，这个name就是布局里面的一个元素节点的名字。

## 6. Factory and Factory2

从上面已经知道，创建view的工作实际上是Factory完成的，那么我们可以看到LayoutInflater里面定义了两个Factory的接口，如下：

{% highlight java %}
public interface Factory {
    /**
     * Hook you can supply that is called when inflating from a LayoutInflater.
     * You can use this to customize the tag names available in your XML
     * layout files.
     * 
     * <p>
     * Note that it is good practice to prefix these custom names with your
     * package (i.e., com.coolcompany.apps) to avoid conflicts with system
     * names.
     * 
     * @param name Tag name to be inflated.
     * @param context The context the view is being created in.
     * @param attrs Inflation attributes as specified in XML file.
     * 
     * @return View Newly created view. Return null for the default
     *         behavior.
     */
    public View onCreateView(String name, Context context, AttributeSet attrs);
}

public interface Factory2 extends Factory {
    /**
     * Version of {@link #onCreateView(String, Context, AttributeSet)}
     * that also supplies the parent that the view created view will be
     * placed in.
     *
     * @param parent The parent that the created view will be placed
     * in; <em>note that this may be null</em>.
     * @param name Tag name to be inflated.
     * @param context The context the view is being created in.
     * @param attrs Inflation attributes as specified in XML file.
     *
     * @return View Newly created view. Return null for the default
     *         behavior.
     */
    public View onCreateView(View parent, String name, Context context, AttributeSet attrs);
}
{% endhighlight %}

看到第一个Factory的注释说明，这是一个渲染view的时候的Hook，我们可以用来自定义tag的名字在xml的布局里面。第二个Factory2则比第一个Factory多了一个parent的参数，关于parent的说明是，创建的view将在parent里面。还有一点需要注意到的是Factory2继承了Factory。

从源码可以知道LayoutInflater有两个属性，Factory和Factory2都有，而且是优先调用Factory2的onCreateView，只有在Factory2为null的时候才调用Factory。

那么Factory的实现类在哪里？它们是什么时候赋值给LayoutInflater的？

我们直接可以查看到Factory2的实现类是Activity。看到源码如下：

{% highlight java %}
/**
 * Standard implementation of
 * {@link android.view.LayoutInflater.Factory#onCreateView} used when
 * inflating with the LayoutInflater returned by {@link #getSystemService}.
 * This implementation does nothing and is for
 * pre-{@link android.os.Build.VERSION_CODES#HONEYCOMB} apps.  Newer apps
 * should use {@link #onCreateView(View, String, Context, AttributeSet)}.
 *
 * @see android.view.LayoutInflater#createView
 * @see android.view.Window#getLayoutInflater
 */
@Nullable
public View onCreateView(String name, Context context, AttributeSet attrs) {
    return null;
}

/**
 * Standard implementation of
 * {@link android.view.LayoutInflater.Factory2#onCreateView(View, String, Context, AttributeSet)}
 * used when inflating with the LayoutInflater returned by {@link #getSystemService}.
 * This implementation handles <fragment> tags to embed fragments inside
 * of the activity.
 *
 * @see android.view.LayoutInflater#createView
 * @see android.view.Window#getLayoutInflater
 */
public View onCreateView(View parent, String name, Context context, AttributeSet attrs) {
    if (!"fragment".equals(name)) {
        return onCreateView(name, context, attrs);
    }

    return mFragments.onCreateView(parent, name, context, attrs);
}
{% endhighlight %}

然后上面的方法并没有具体实现，正常来说是返回null，并且是用于处理`<fragment>`标签的。继续查看LayoutInflater的源码，发现给mFactory2赋值有两个地方，一个是LayoutInflater的构造，另外一个是setter方法。

{% highlight java %}
protected LayoutInflater(LayoutInflater original, Context newContext) {
    mContext = newContext;
    mFactory = original.mFactory;
    mFactory2 = original.mFactory2;
    mPrivateFactory = original.mPrivateFactory;
    setFilter(original.mFilter);
}
{% endhighlight %}

由于我们看不到`cloneInContext()`，也没法知道BaseContext的LayoutInflater是如何实例化的，猜测调用cloneInContext的时候会调用这个构造方法，然后传了原来的Factory。至于Base Context的Factory则是通过setter方法来设置的。

关于两个setter方法如下：

{% highlight java %}
/**
 * Attach a custom Factory interface for creating views while using
 * this LayoutInflater.  This must not be null, and can only be set once;
 * after setting, you can not change the factory.  This is
 * called on each element name as the xml is parsed. If the factory returns
 * a View, that is added to the hierarchy. If it returns null, the next
 * factory default {@link #onCreateView} method is called.
 * 
 * <p>If you have an existing
 * LayoutInflater and want to add your own factory to it, use
 * {@link #cloneInContext} to clone the existing instance and then you
 * can use this function (once) on the returned new instance.  This will
 * merge your own factory with whatever factory the original instance is
 * using.
 */
public void setFactory(Factory factory) {
    if (mFactorySet) {
        throw new IllegalStateException("A factory has already been set on this LayoutInflater");
    }
    if (factory == null) {
        throw new NullPointerException("Given factory can not be null");
    }
    mFactorySet = true;
    if (mFactory == null) {
        mFactory = factory;
    } else {
        mFactory = new FactoryMerger(factory, null, mFactory, mFactory2);
    }
}

/**
 * Like {@link #setFactory}, but allows you to set a {@link Factory2}
 * interface.
 */
public void setFactory2(Factory2 factory) {
    if (mFactorySet) {
        throw new IllegalStateException("A factory has already been set on this LayoutInflater");
    }
    if (factory == null) {
        throw new NullPointerException("Given factory can not be null");
    }
    mFactorySet = true;
    if (mFactory == null) {
        mFactory = mFactory2 = factory;
    } else {
        mFactory = mFactory2 = new FactoryMerger(factory, factory, mFactory, mFactory2);
    }
}
{% endhighlight %}

看到上面的注释说明，通过mFactorySet这个标记来控制这个setter方法只能被调用一次，并且不可以传null参数进来。

到这里还是没有发现Factory2的实现类，于是思考到上面的注释是说Factory只是一个Hook，也就是没有具体实现也是可以的。回头继续查看`createViewFromTag()`的代码，发现以下逻辑：

{% highlight java %}
View view;
if (mFactory2 != null) {
    view = mFactory2.onCreateView(parent, name, viewContext, attrs);
} else if (mFactory != null) {
    view = mFactory.onCreateView(name, viewContext, attrs);
} else {
    view = null;
}

if (view == null && mPrivateFactory != null) {
    view = mPrivateFactory.onCreateView(parent, name, viewContext, attrs);
}

if (view == null) {
    final Object lastContext = mConstructorArgs[0];
    mConstructorArgs[0] = viewContext;
    try {
        if (-1 == name.indexOf('.')) {
            view = onCreateView(parent, name, attrs);
        } else {
            view = createView(name, null, attrs);
        }
    } finally {
        mConstructorArgs[0] = lastContext;
    }
}

return view;
{% endhighlight %}

上面正是完美诠释了Hook的意义，首先会判断mFactory2，如果为空则使用mFactory。就算使用Factory创建了View，如果为null，则判断mPrivateFactory是否存在，再次创建一个View。如果所有Factory创建的View都是null，则会调用到自身的方法`onCreateView()`或者`createView()`了，实际上，最终都是调用到`createView()`这个方法。查看这个方法的说明如下：

{% highlight java %}
/**
 * Low-level function for instantiating a view by name. This attempts to
 * instantiate a view class of the given <var>name</var> found in this
 * LayoutInflater's ClassLoader.
 * 
 * <p>
 * There are two things that can happen in an error case: either the
 * exception describing the error will be thrown, or a null will be
 * returned. You must deal with both possibilities -- the former will happen
 * the first time createView() is called for a class of a particular name,
 * the latter every time there-after for that class name.
 * 
 * @param name The full name of the class to be instantiated.
 * @param attrs The XML attributes supplied for this instance.
 * 
 * @return View The newly instantiated view, or null.
 */
public final View createView(String name, String prefix, AttributeSet attrs)
        throws ClassNotFoundException, InflateException {
        //...
}
{% endhighlight %}

显然上面注释所说这是一个低级的方法，用于同于节点的名字来实例化一个view，实际上就是通过View的类名来创建了实例，并且给这些View对象设置属性值，实际上传的是AttributeSet对象给View，也就是为什么我们自定义View的时候会有构造方法里面传来了AttributeSet。

到这里LayoutInflater就把整个xml布局文件转换成了View的对象了。回到最初的一个疑问，为什么叫Inflater，而不是Render？我们说渲染的意思是把图形显示在屏幕上，而这里，并非把xml显示在了屏幕上面了，实际上，一个xml布局定义了界面，然后Inflater只是把xml的布局文件转化成了View的对象，只是java里面的对象，真实如何把这些对象渲染到屏幕还是底层的系统完成的，那里涉及到了OpenGLES等更多的知识了。

## 7. 运行切换皮肤和主题

如果app有day/night模式，或者可以切换皮肤的功能，那么就需要用到了这里的知识了。

**首先总结一下，所有的xml布局文件转换成View对象都是通过LayoutInflater的，而LayoutInflater里面依赖了XmlPullParser来解析编译的xml文件，然后调用了LayoutInflater的方法createView来创建View对象，LayoutInflater还提供了两个Hook，即Factory和Factory2来给开发者自定义修改创建的View对象。每当实例化一个View对象的时候都需要调用一次Factory2的方法，我们只需要调用LayoutInflater的createView()方法来创建对象，然后修改属性即可。**

因此，我们可以做的工作就是自定义Factory2的实现类，在Activity的setContentView之前把这个SkinFactory设置到Inflater里面去即可。关于如何有效实现切换皮肤或者主题的Factory，网上很多文章了，这里就不再搬砖，只是提一下这个实现。关键在于理解了Android的View是如何从xml转化而来的原理即可，剩下的便是写代码的逻辑了。