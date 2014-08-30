---
layout: post
title:  "Android的ViewPager学习笔记!"
date:   2014-07-30 07:30:00
categories: android
type: android
---

viewpager可以实现屏幕滑动的效果，用来做画廊，启动界面这些都不错，而且比Flipper，Gallery都快，顺畅。可以考虑把tongli的那个图片换成viewpager。

**官方的viewpager使用很简单：**    
首先在布局设置viewpager控件，然后重写PagerAdapter，里面的data是List<View>即可，主要重载两个方法：  
{% highlight java %}
@Override
public void destroyItem(ViewGroup container, int position, Object object) {
	((ViewPager)container).removeView(data.get(position));
}

@Override
public Object instantiateItem(ViewGroup container, int position) {
	((ViewPager)container).addView(data.get(position));
	return data.get(position);
}
{% endhighlight %}
viewpager可以设置setOnPageChangeListener事件。

一般结合viewpager使用的是在顶部有一些tab，当滑动viewpager的时候，tab也跟着变化。  
官方的有这两个控件，他们都是作为viewpager的子控件使用：  
PagerTabStrip: 交互式  
PagerTitleStrip: 非交互式  

PagerTabStrip：  
1.点击上面的标题可以实现ViewPager的切换。  
2.选中的文字下方包含指引线  
3.显示全宽下划线（setDrawFullUnderline）

PagerTitleStrip：  
1.点击上面的标题无反应。  
2.无上述描述。

代码上没多少区别，vviewpager和strip无需再绑定了，只要在上面的adapter里面再重写一个方法和设置title的data  
{% highlight java %}
@Override  
public CharSequence getPageTitle(int position)   {  
	 return Titles.get(position);  
}
{% endhighlight %}
另外，strip有很多属性可以设置的。

**FragmentPagerAdapter**  
viewpager的adapter的另外一种写法，不是继承PagerAdapter，而是继承FragmentPagerAdapter，没什么难的，只是viewpager的每一个view都是fragment而已。而且，这个adapter还需要传FragmentManager进去。

有一个开源的strip，那些tap是可以滑动的：  
pagerslidingtabstrip：http://blog.csdn.net/top_code/article/details/17438027
如果要用它的时候再认真去学，公司就是用这个的，然后再适当修改为自己需求的。

比较成熟和出名的是这个Android-ViewPagerIndicator:http://viewpagerindicator.com/

网上还有人用actionbar来实现：  
http://blog.csdn.net/qinyuanpei/article/details/17837165
http://blog.csdn.net/eclipsexys/article/details/8688538

也有人自定义：http://blog.csdn.net/jdsjlzx/article/details/20937985

以上有机会再去学习。。。

**其他一些设置**  
viewpager去掉边缘滑动阴影（这些效果耗内存）：
android:overScrollMode="never"
