---
layout: post
title:  "Android的View学习"
date:   2014-07-27 10:00:03
categories: android
type: android
---

## 

引用网上一句话：Android系统中的所有UI类都是建立在View和ViewGroup这两个类的基础上的。所有View的子类成为”Widget”，所有ViewGroup的子类成为”Layout”。  
View和ViewGroup之间采用了组合设计模式，可以使得“部分-整体”同等对待。ViewGroup作为布局容器类的最上层，布局容器里面又可以有View和ViewGroup。

View是最顶层的界面类，ViewGroup是继承View的抽象类，所以ViewGroup是一组view的集合。view又是所有界面的父类引用，view引用可以指向viewgroup，实现了多态，
在adapter那里getView什么的是很好的体现。

## Canvas

当自定义一个控件后，可以给这个控件自定义更多属性，例如我自定义一个ImageView，给它一个属性，是否圆角。

**第一步**  
在res/values下创建attrs.xml文件，里面定义好属性名称和类型:  
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

**第二步**
编写自定义控件类

**第三步**
在布局文件里面引用新的命名空间。
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

**第四步**  
在自定义控件类里面的构造方法获取这个属性值：
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
获得了属性，就可以自己去构造这个view了。  
注意：  
属性的命名规则是：采取的名字_属性这种格式。例如：R.styleable.roundedimageview_border_thickness  
还有：  
一定要执行typeArray.recycle();这是回收给别的地方用。

*Switch*  
这个控件是4.0以后才有的，如果想要低版本用，可以直接拿官网的源码来用。。。。这也说明了一点，其实如果有时候一些新版本的控件也可以这样拿来用。