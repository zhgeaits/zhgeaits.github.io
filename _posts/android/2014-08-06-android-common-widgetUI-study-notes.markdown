---
layout: post
title:  "Android的常用UI学习笔记&会持续更新的!"
date:   2014-04-05 11:00:03
categories: android
type: android
---

TextView可以设置这样一个属性，文字上面配一个图片。
{% highlight xml %}
android:drawableTop="@drawable/default_tip"
{% endhighlight %}

TextView可以设置文字对齐，但是排版问题解决不了。。。
{% highlight xml %}
android:gravity="right"
{% endhighlight %}

给一个TextView设置Text后，如何马上获得这个TextView的宽度？

给一个View动态在代码里面设置android:layout_toLeftOf这样的属性，这个在RelativeLayout里面的属性：  

{% highlight java %}
RelativeLayout.LayoutParams params = new RelativeLayout.LayoutParams(
                    ViewGroup.LayoutParams.WRAP_CONTENT,
                    ViewGroup.LayoutParams.WRAP_CONTENT);
params.addRule(RelativeLayout.RIGHT_OF, R.id.sign_label);
params.addRule(RelativeLayout.CENTER_VERTICAL);
params.rightMargin = dip2px(this, 30);
view.setLayoutParams(params);
{% endhighlight %}

dp和pix之间的转换：  
{% highlight java %}
public static int dip2px(Context context, float dpValue) {
	final float scale = context.getResources().getDisplayMetrics().density;
	return (int) (dpValue * scale + 0.5f);
}

public static int px2dip(Context context, float pxValue) {
	final float scale = context.getResources().getDisplayMetrics().density;
	return (int) (pxValue / scale + 0.5f);
}
{% endhighlight %}