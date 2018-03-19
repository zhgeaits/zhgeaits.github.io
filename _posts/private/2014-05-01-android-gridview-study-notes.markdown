---
layout: post
title:  "Android的GridView学习笔记!"
date:   2014-05-01 11:00:03
categories: private
type: private
---

Adapter也是继承BaseAdapter。

gridview去掉边缘滑动阴影（这些效果耗内存）：
{% highlight xml %}
android:overScrollMode="never"
{% endhighlight %}

根据每一项的高度来动态设置GridView的高度  
{% highlight java %}
public static void setGridViewHeightBasedOnChildren(GridView gridView, Context context, int itemHeight) {

	ListAdapter listAdapter = gridView.getAdapter();

	if (listAdapter == null) {
		return;
	}

	int totalHeight = 0;
	int lines = 2;
	int numColumns = gridView.getNumColumns();
	if(listAdapter.getCount() % numColumns == 0) {
		lines = listAdapter.getCount() / numColumns;
	} else {
		lines = listAdapter.getCount() / numColumns + 1;
	}
	for (int i = 0; i < lines; i++) {
		totalHeight += CommonUtils.dip2pixel(context, itemHeight) + gridView.getVerticalSpacing();
	}

	ViewGroup.LayoutParams params = gridView.getLayoutParams();
	params.height = totalHeight;
	gridView.setLayoutParams(params);
	gridView.requestLayout();
}
{% endhighlight %}