---
layout: post
title:  "Android的ListView学习笔记!"
date:   2014-03-15 11:00:03
categories: android
type: android
---

listview 每个item的高度：发现如果listview的每一项item没有图片的话，不管怎么设置item的布局高度都是无效的，网上查过以后，发现在item的布局里面设置最小高度就可以了。
{% highlight xml %}
android:minHeight="50dp"
{% endhighlight %}

listview去掉边缘滑动阴影和按下颜色（这些效果耗内存）：
{% highlight xml %}
android:overScrollMode="never"
android:listSelector="@android:color/transparent"
{% endhighlight %}
如果要定制按下的颜色变化，就需要自己写一个selector，这里会有点问题，如果每个item本来就有背景颜色的，然后selector只是写成了这样：  
{% highlight xml %}
<?xml version="1.0" encoding="utf-8"?>
<selector xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:drawable="@color/item_press" android:state_pressed="true"/>
    <item android:drawable="@color/item_normal"/>
</selector>
{% endhighlight %}
这个selector对于正常的点击是好用的，但是因为listview的item有很多状态：  
默认时的背景图片：什么都不设置  
没有焦点时的背景图片：android:state_window_focused="false"   
非触摸模式下获得焦点并单击时的背景图片：android:state_focused="true" android:state_pressed="true"   
触摸模式下单击时的背景图片：android:state_focused="false" android:state_pressed="true"
选中时的图片背景：android:state_selected="true"  
获得焦点时的图片背景：android:state_focused="true"   

所以如果还是用上面的selector的话就会有问题，默认的颜色和item的背景重叠了。修改如下问题就解决了：  
{% highlight xml %}
<?xml version="1.0" encoding="utf-8"?>
<selector xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:drawable="@color/shenqu_local_upload_press" android:state_pressed="true"/>
    <item android:drawable="@color/transparent" android:state_window_focused="false"/>
    <item android:drawable="@color/transparent" android:state_focused="true" android:state_pressed="true"/>
    <item android:drawable="@color/shenqu_local_upload_normal"/>
</selector>
{% endhighlight %}

listview作为聊天界面的时候，要自动滚到底部，可以设置一个定时器，然后执行chatList.setSelection(fragmentAdapter.getCount() - 1);就可以。  
setSelection是直接跳过去，而smoothScrollToPosition是滚过去。

遇到一个奇怪问题，在聊天界面，聊天内容如果用的是listview的话，点击输入框弹出键盘以后，listview会自动滚到底部，但是公司的是用PullToRefreshView，虽然是继承listview的，但是就不会滚动到底部，试过用上面的定时器方法，但是时间不好控制。也试过重写Layout来捕获OnResize方法，但是竟然还是无效。。。后来在stackoverflow上面找到了，加入这个属性就行：  
android:transcriptMode="alwaysScroll"  
listview默认应该是normal，估计PullToRefreshView做了手脚。

分割线：  
android:divider="@null";
android:divider="@drawable/listview_horizon_line"

每项的间距：
android:dividerHeight="10dp"


给listview设置一个黏住，浮起来不动的标题：StickyListHeaders  
github开源：https://github.com/emilsjolander/StickyListHeaders

根据每一项的高度来动态设置listview的高度  
{% highlight java %}
public static void setListViewHeightBasedOnChildren(ListView listView, Context context, int itemHeight) {

	ListAdapter listAdapter = listView.getAdapter();

	if (listAdapter == null) {
		return;
	}

	int totalHeight = 0;
	for (int i = 0; i < listAdapter.getCount(); i++) {
		totalHeight += CommonUtils.dip2pixel(context, itemHeight) + listView.getDividerHeight();
	}

	ViewGroup.LayoutParams params = listView.getLayoutParams();
	params.height = totalHeight;
	listView.setLayoutParams(params);
	listView.requestLayout();
}
{% endhighlight %}

getView()和notifyDataSetChanged()

一般调用notifyDataSetChanged()就会触发getView()方法。然后我在项目这里发现了不管怎么调用notifyDataSetChanged都不会触发getView，想了一天都不能解决，后来才发现，必须在UI线程调用啊。。。。。。