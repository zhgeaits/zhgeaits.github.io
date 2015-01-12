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

**FragmentStatePagerAdapter**  
FragmentPagerAdapter更多的用于少量界面的ViewPager，比如Tab。划过的fragment会保存在内存中，尽管已经划过。而FragmentStatePagerAdapter和ListView有点类似，会保存当前界面，以及下一个界面和上一个界面（如果有），最多保存3个，其他会被销毁掉。

**FixedFragmentStatePagerAdapter**  
可以google一下这个，说是修复被回收后重启时不能从fragmentstatepageradapter恢复的bug。但是我还是不太懂，先记录下来，也许以后遇到这个问题就懂了。

**setOffscreenPageLimit**  
设置mPager.setOffscreenPageLimit(3);viewpager的适配器会预装在3个view,包含当前显示的view，一共4个，这是一个窗口,移动窗口时候，后面加载一个，前面销毁一个，最好能重写adapter的onDestoryItem方法。如果fragment缓存了的话，就不会去执行getItem方法，就会不去new Fragment了。

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

* **设置自动滑动的切换速度**  
由于公司项目有需求，viewpager加一个按钮点击，然后滑动到下一个页面，直接去调用：mViewPager.setCurrentItem(mPos, true);发现切换得很快。调查发现，可以用反射修改viewpager的scroller属性。  
Field sField = ViewPager.class.getDeclaredField("mScroller");  
sField.setAccessible(true);  
mScroller = new FixedSpeedScroller(mViewPager.getContext(), mDuration);  
sField.set(mViewPager, mScroller);  
FixedSpeedScroller在我的github上有。。 
给viewpager设置onTouch事件，如果返回true则可以屏蔽viewpager手动滑动。

* **滑动页面时对按钮进行渐变效果**  
例子如下：
{% highlight java %}
//viewpager页面改变事件，主要做的是对开始按钮和下一步箭头按钮进行透明度处理
mViewPager.setOnPageChangeListener(new ViewPager.OnPageChangeListener() {
	@Override
	public void onPageScrolled(int position, float positionOffset, int positionOffsetPixels) {
		//在第三页滑到第四页（最后页），马上体验按钮要逐渐进行显示，下一步按钮逐渐消失，通过设置Alpha值来改变透明度
		//positionOffset变化是0-1，向后滑逐渐增大，向前逐渐变小。
		if (position >= 2 && positionOffset != 0) {
			begin.setVisibility(View.VISIBLE);
			next.setVisibility(View.VISIBLE);
			if (positionOffset > 0.9) {
				//大于0.9以后则直接显示begin，消失next
				begin.getBackground().setAlpha(255);
				next.getBackground().setAlpha(0);
			} else if (positionOffset > 0) {
				//只要是大于0，那么就进行一个渐变效果
				begin.getBackground().setAlpha((int) (255 * positionOffset));
				next.getBackground().setAlpha(255 - (int) (255 * positionOffset));
			} else if (positionOffset == 0) {
				//为0情况，则next消失
				next.setVisibility(View.GONE);
			}
		} else if (position != 3) {
			//不是最后一页情况
			begin.setVisibility(View.GONE);
			next.setVisibility(View.VISIBLE);
		}
		mPos = position;
	}

	@Override
	public void onPageSelected(int position) {
		//如果是最后一页，则进行显示 马上体验按钮，隐藏 下一步箭头按钮
		//其他页相反处理
		if (position == 3) {
			begin.setVisibility(View.VISIBLE);
			next.setVisibility(View.GONE);
		} else {
			begin.setVisibility(View.GONE);
			next.setVisibility(View.VISIBLE);
		}
		mPos = position;
	}

	@Override
	public void onPageScrollStateChanged(int state) {
	}
});
{% endhighlight %}

**要拿到viewpager的指定或者当前子view方法**  
方法一：  
创建子view的时候设置一个ID，然后可以根据这个id来find。  
{% highlight java %}
view.setId(index);
mPager.findViewById(position);
{% endhighlight %}  

方法二：  
在adapter里面设置。  
{% highlight java %}
	private View mCurrentView;

	@Override
	public void setPrimaryItem(ViewGroup container, int position, Object object) {
		mCurrentView = (View)object;
	}

	public View getPrimaryItem() {
		return mCurrentView;
	}
{% endhighlight %}

方法一在viewpager的onpagechange事件里面使用很好，方法二就不行了，因为是会先处理onpagechange事件，再调用adapter里面的方法来设置。注意，mPager.childAt(mPager.getCurrentItem());这个方法不准确，viewpager超过三个就不行了。