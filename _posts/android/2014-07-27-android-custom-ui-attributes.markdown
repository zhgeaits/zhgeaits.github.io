---
layout: post
title:  "Android的自定义控件属性attrs.xml, TypedArray学习笔记!"
date:   2014-07-27 10:00:03
categories: android
type: android
---

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
xmlns:zg="http://schemas.android.com/apk/res/org.zhangge.ui.image" 
{% endhighlight %}
org.zhangge.ui.image这个是我的控件类所在包名

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
