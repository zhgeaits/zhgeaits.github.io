---
layout: post
title:  "Android的Canvas类, Paint类, Color类, ColorMatrix类, Matrix类学习笔记!"
date:   2014-08-10 11:00:03
categories: android
type: android
---

**Canvas**

Canvas是画布的意思，没有深入了解过这个类，不知道是不是一块内存，不过，可以理解为一块显示区域，因为，画布嘛，不就是给人看的么，内存又不是给人看的，所以，bitmap才是一块内存，包含像素，要把bitmap画到canvas上面去，当然还可以画很多元素。

一般，在View里面的onDraw方法都带有一个canvas，可以用来画东西。调用invalidate()方法会触发onDraw()方法。

更多canvas的方法：  
public void drawArc (RectF oval, float startAngle, float sweepAngle, boolean useCenter, Paint paint);  
oval是圆所在的矩阵，startAngle是起始角度，sweepAngle是弧的角度，useCenter是否显示半径连线，即是否画到圆心，最后是paint画笔。

**Paint**

Paint是画笔的意思，其实画笔就是带有颜色(Color)和样式(Styles)这些属性。调用canvas画东西的时候，必须传入一个画笔。画笔还可以设置锯齿，argb，字体大小，边框等属性。

更多Paint的方法：  
paint.setAntiAlias(true);  //消除锯齿

**Color**

Color就是颜色类，里面定义不少常用颜色的常量值，也有不少好用的方法，例如分别可以提取argb的值。

**ColorMatrix**

ColorMatrix是颜色矩阵，非常厉害的类，理解可能比较困难，可以参考官网：  
http://developer.android.com/reference/android/graphics/ColorMatrix.html   
可以通过矩阵相乘来改变颜色值，这个矩阵是4x5的，乘以颜色向量[R,G,B,A]，矩阵乘法还记得吧，所以修改矩阵就可以修改argb了。可以参考我的RBPlayer里面的colorview类。这个的类的应用一般用在图片美化方面，改变图片的样式，泛黄啊，变灰啊什么的。

**Matrix**

Matrix是修改位置，例如进行旋转，平移什么的，也非常强大，目前还没用到过，但是有一些比较厉害的应用就是实现Folding Layout，可以去Google一下。。。
