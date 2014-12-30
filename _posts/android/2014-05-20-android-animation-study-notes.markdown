---
layout: post
title:  "Android的动画Animation学习笔记!"
date:   2014-05-20 11:00:03
categories: android
type: android
---

由于没有认真做过关于动画的东西，所以记录内容也比较少，只是记录了一些基本的了解而已，以后慢慢接触动画时候再一点点补充。  
官网：http://developer.android.com/guide/topics/graphics/overview.html  
http://developer.android.com/guide/topics/resources/animation-resource.html  
eoe上面的学习地址：http://www.eoeandroid.com/thread-173194-1-1.html  
至于书上的介绍学习就比较少了。

android的动画主要分这几种：  
补间动画：名字难以理解。它作用于view，定义一系列关于位置，大小，旋转和透明度的改变。有缺陷，如果view
的形状改变，那么控件（如按钮）的按事件触发位置还在老位置。  
帧动画：就是一帧帧的播放图片。  
属性动画 ：可以用于任何地方，且没有补间动画的缺陷。

**补间动画**  
只要在res/anim下创建动画的资源文件，根节点是<set>，然后主要的四个节点是<alpha><scale> <translate> <rotate>，
四个节点分别是透明度，尺寸，位置和旋转，具体可以看官网介绍。当动画开始的时候，view是会根据这四个设置的属性进行变化。

当然，可以在xml文件设置，也可以在代码动态设置动画，四个节点分别对应AlphaAnimation。。。等等。  

使用：  
使用Animation anima = AnimationUtils.loadAnimation(Context, resourceId);加载xml定义的动画，然后view.startAnimation(anima)即可。可以start也可stop的。。  
如果不是xml定义的，是分别new的AlphaAnimation这些，就分别start即可。

scale:    
interpolator指定动画插入器，常见的有加速减速插入器accelerate_decelerate_interpolator，加速插入器accelerate_interpolator，减速插入器decelerate_interpolator。  
fromXScale,fromYScale，动画开始前X,Y的缩放，0.0为不显示，1.0为正常大小  
toXScale，toYScale，动画最终缩放的倍数，1.0为正常大小，大于1.0放大  
pivotX，pivotY动画起始位置，相对于屏幕的百分比,两个都为50%表示动画从屏幕中间开始  
startOffset，动画多次执行的间隔时间，如果只执行一次，执行前会暂停这段时间，单位毫秒  
duration，一次动画效果消耗的时间，单位毫秒，值越小动画速度越快   
repeatCount，动画重复的计数，动画将会执行该值+1次  
repeatMode，动画重复的模式，reverse为反向，当第偶次执行时，动画方向会相反。restart为重新执行，方向不变  
fillBefore是指动画结束时画面停留在此动画的第一帧;  
fillAfter是指动画结束是画面停留在此动画的最后一帧。  


**帧动画**  
也是定义xml文件，书上说必须放在res/drawable/下，但是网上说放在res/anim/下，我测试了，都可以。xml文件的根节点是<animation-list>，下面是<item>节点，每个item放一张图片。帧动画是绑定到imageview的，这样使用：  
imageview.setBackgroundResource(R.drawable.test);    
imageviewAnima = (AnimationDrawable) imageview.getBackground();    
imageviewAnima.start();     
下面这种用法已经过时：  
animate = (AnimationDrawable) getResources().getDrawable(R.anim.test);   
imageview.setBackgroundDrawable(animate);    
animate.start();    
注意的是：不能在oncreate里面start，因为imageview还没初始化好。  

**属性动画**  
发现一般补间动画就够用了，属性动画看UI那本书的例子就可以。属性动画就是改变view的属性，在一定范围的变化的过程中进行改变。    
到官网查看一下就知道，不要使用ObjectAnimator,老是有很多属性没效果，使用ValueAnimator,自己的例子如下：  
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

学习资源：  
http://blog.csdn.net/singwhatiwanna/article/details/17841165


**Activity动画**  
要修改启动activity和关闭activity的动画，就在startActivity的时候调用overridePendingTransition(R.anim.in, R.anim.out); 在finish()的时候也可以调用。  
activity有新的api来使用动画，ActivityOptions，看下面的blog即可。  
http://www.cnblogs.com/tianzhijiexian/p/4087917.html  
http://www.cnblogs.com/tianzhijiexian/p/4128045.html


**viewpager动画**  
实现这个接口：PageTransformer  
http://www.csdn123.com/html/topnews201408/88/5988.htm