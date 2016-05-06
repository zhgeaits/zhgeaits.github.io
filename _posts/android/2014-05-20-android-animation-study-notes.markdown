---
layout: post
title:  "Android的动画学习"
date:   2014-05-20 11:00:03
categories: android
supertype: career
type: android
---

## 1. 引言

动画知识不难理解，网上学习的资料也很多，[官网1](http://developer.android.com/guide/topics/graphics/overview.html)、[官网2](http://developer.android.com/guide/topics/resources/animation-resource.html)、[eoe上面的学习地址](http://www.eoeandroid.com/thread-173194-1-1.html)、[学习资源](http://blog.csdn.net/singwhatiwanna/article/details/17841165)。

还有一个开源动画库：nineoldandroids，在Github上开源的，它是兼容低版本的。

android的动画主要分这几种：

- 补间动画：名字难以理解，英文名字叫Tween Animation。它作用于view，定义一系列关于位置，大小，旋转和透明度的改变。有缺陷，如果view的形状改变，那么控件（如按钮）的按事件触发位置还在老位置。
- 帧动画：就是一帧帧的播放图片。
- 属性动画 ：可以用于任何地方，且没有补间动画的缺陷。

## 2. 补间动画

只要在res/anim下创建动画的资源文件，根节点是<set>，然后主要的四个节点是<alpha>、<scale>、<translate>、<rotate>，四个节点分别是透明度，尺寸，位置和旋转，具体可以看官网介绍。当动画开始的时候，view是会根据这四个设置的属性进行变化。

当然，可以在xml文件设置，也可以在代码动态设置动画，四个节点分别对应AlphaAnimation...等等。  

### 2.1 使用

加载xml定义的动画:

>Animation anima = AnimationUtils.loadAnimation(Context, resourceId);

然后`view.startAnimation(anima)`即可。可以start没有stop的，只有clearAnimartion()。

如果不是xml定义的，是分别new的AlphaAnimation这些，就分别start即可。如果在webview里面用补间动画，有时候会花屏，这时候就需要换成帧动画或者属性动画了。  

### 2.2 关于scale的属性

- interpolator：指定动画插入器，常见的有加速减速插入器accelerate_decelerate_interpolator，加速插入器accelerate_interpolator，减速插入器decelerate_interpolator。  
- fromXScale和fromYScale：动画开始前X,Y的缩放，0.0为不显示，1.0为正常大小  
- toXScale和toYScale：动画最终缩放的倍数，1.0为正常大小，大于1.0放大  
- pivotX，pivotY：动画起始位置，相对于屏幕的百分比,两个都为50%表示动画从屏幕中间开始  
- startOffset：动画多次执行的间隔时间，如果只执行一次，执行前会暂停这段时间，单位毫秒  
- duration：一次动画效果消耗的时间，单位毫秒，值越小动画速度越快   
- repeatCount：动画重复的计数，动画将会执行该值+1次  
- repeatMode：动画重复的模式，reverse为反向，当第偶次执行时，动画方向会相反。restart为重新执行，方向不变  
- fillBefore：是指动画结束时画面停留在此动画的第一帧;  
- fillAfter：是指动画结束是画面停留在此动画的最后一帧。  

### 2.3 配置activity的启动和关闭动画

要修改启动activity和关闭activity的动画，就在startActivity或者finish的时候调用：

> overridePendingTransition(R.anim.push_left_in, R.anim.push_left_out);

也可以配置全局的，首先配置theme，然后配置为application就OK，如下：

{% highlight xml %}

<style name="AppTheme" parent="@android:style/Theme.NoTitleBar">  
    <!-- 设置没有标题 -->  
    <item name="android:windowNoTitle">true</item>  
    <!-- 设置activity切换动画 -->  
    <item name="android:windowAnimationStyle">@style/activityAnimation</item>  
</style>  

<!-- animation 样式 -->  
<style name="activityAnimation" parent="@android:style/Animation">  
    <item name="android:activityOpenEnterAnimation">@anim/push_left_in</item>  
    <item name="android:activityOpenExitAnimation">@anim/push_left_out</item>
    <item name="android:activityCloseEnterAnimation">@anim/push_right_in</item>
    <item name="android:activityCloseExitAnimation">@anim/push_right_out</item> 
</style>

{% endhighlight %}

### 2.4 activity动画的新接口实现

上面的接口效率很低的，于是出了新接口高效率使用动画，如下：

{% highlight java %}

ActivityOptionsCompat options = ActivityOptionsCompat.makeCustomAnimation(context,
                R.anim.slide_in_from_right, R.anim.slide_out_from_left);
ActivityCompat.startActivity((Activity) context, intent, options.toBundle());

int[] location = new int[2];
view.getLocationOnScreen(location);
int x = location[0];
int y = location[1];
ActivityOptionsCompat options = ActivityOptionsCompat
    .makeScaleUpAnimation(view, x/2, y/2, view.getMeasuredWidth(), view.getMeasuredHeight());
ActivityCompat.startActivity((Activity)context, intent, options.toBundle());

{% endhighlight %}

ActivityOptionsCompat是一个兼容性接口，可以看更多ActivityOptions。[资料1](http://www.cnblogs.com/tianzhijiexian/p/4087917.html)、[资料2](http://www.cnblogs.com/tianzhijiexian/p/4128045.html)。

## 3. 帧动画

也是定义xml文件，书上说必须放在res/drawable/下，但是网上说放在res/anim/下，我测试了，都可以。xml文件的根节点是<animation-list>，下面是<item>节点，每个item放一张图片。帧动画是绑定到imageview的，这样使用：

{% highlight java %}

imageview.setBackgroundResource(R.drawable.test);    
imageviewAnima = (AnimationDrawable) imageview.getBackground();    
imageviewAnima.start();     
//下面这种用法已经过时：  
animate = (AnimationDrawable) getResources().getDrawable(R.anim.test);   
imageview.setBackgroundDrawable(animate);    
animate.start(); 

{% endhighlight %}

注意的是：不能在oncreate里面start，因为imageview还没初始化好。  

## 4. 属性动画

发现一般补间动画就够用了，属性动画看UI那本书的例子就可以。属性动画就是改变view的属性，在一定范围的变化的过程中进行改变。到官网查看一下就知道，不要使用ObjectAnimator，老是有很多属性没效果，使用ValueAnimator,自己的例子如下：

{% highlight java %}

ValueAnimator animator = ValueAnimator.ofInt(mLoginContainerHeight, 0);
animator.setDuration(300).start();
animator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
	@Override
	public void onAnimationUpdate(ValueAnimator animation) {
		int value = (Integer) animation.getAnimatedValue();
		mLoginContainer.getLayoutParams().height = value;
		mLoginContainer.requestLayout();
	}

});

{% endhighlight %}

## 5 其他

### 5.1 ViewPager动画

[实现这个接口：PageTransformer](http://www.csdn123.com/html/topnews201408/88/5988.htm)
