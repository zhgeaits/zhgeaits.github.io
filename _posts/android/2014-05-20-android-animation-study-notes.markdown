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

**帧动画**  
也是定义xml文件，书上说必须放在res/drawable/下，但是网上说放在res/anim/下，我测试了，都可以。  
xml文件的根节点是<animation-list>，下面是<item>节点，每个item放一张图片。  
帧动画是绑定到imageview的，这样使用：  
imageview.setBackgroundResource(R.drawable.test);  
imageviewAnima = (AnimationDrawable) imageview.getBackground();  
imageviewAnima.start();  
注意的是：不能在oncreate里面start，因为imageview还没初始化好。

**属性动画**  
发现一般补间动画就够用了，属性动画看UI那本书的例子就可以。现在还没用到，以后再补充。  
