---
layout: post
title:  "Android UI View使用大集合[第二波]"
date:   2017-04-11 11:00:03
categories: android
supertype: career
type: android
---

>以前就记录过一次[Android一些UI的使用](http://zhgeaits.me/android/2014/04/05/android-common-widgetUI-study-notes.html)，由于一些新出的UI比较好用，所以需要学习，提高app的性能，并且，还有很多UI View需要学习的。所以再来记录第二波!

## 1. GridView

这是网格View，其实就是多列的listview，而且使用上跟listview几乎一样，所以很多特性技巧什么的可以直接看listview的相关[blog记录](http://zhgeaits.me/android/2014/04/05/android-common-widgetUI-study-notes.html)。

使用上和listview的流程一样，在布局添加一个标签，然后写Adapter，也是继承BaseAdapter，里面的写法，还有item的布局都是一样的，没有多少的区别。

{% highlight xml %}

<GridView
    android:id="@+id/bill_upload_traces_grid"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:overScrollMode="never"
    android:listSelector="@android:color/transparent"
    android:columnWidth="60dp"
    android:gravity="center"
    android:horizontalSpacing="8dp"
    android:verticalSpacing="8dp"
    android:numColumns="4"
    android:stretchMode="columnWidth"/>

{% endhighlight %}

相关属性字段：

android:numColumns="4"   这个是自定列数，也可以是"auto_fit"。

android:columnWidth="60dp"  每列的宽度，也就是Item的宽度，这个和item的宽度是有区别的，注意。

android:horizontalSpacing="8dp"  横向宽度

android:verticalSpacing="8dp"  纵向宽度

android:stretchMode="columnWidth"   缩放与列宽大小同步，这个不好理解。

## 2. ConstraintLayout

这是Android里面很重要的一个布局，去年就已经出来了，但是刚出来的版本一般都很多坑，而且普及不起来，还很依赖于AS。现在AS已经发布2.3版本了，这个已经相对很稳定了，坑也比较少了。从这篇[文章](https://mp.weixin.qq.com/s?__biz=MzI1NTU4MjU3OA==&mid=2247483892&idx=1&sn=158b44ebe7ce42741227cf613750268f&chksm=ea3289e9dd4500ff384443b57973f65c1de6cdb9465f88a883efac29cd930ea5e44fbe72fe91&mpshare=1&scene=1&srcid=04123IVPi1nblZy3vp6O8eN3&key=84515918b5a0dcb41b7d259ced6e1a24a33c41f09901f8c37b3b994e37f686edb1c2e95d86c9db2b82db19b52903bfd622b12c7bd03ddafb97005146c2ced2af730062fcb90244f27aa21b63719e87a1&ascene=0&uin=MTMxNTU3NzYyMA%3D%3D&devicetype=iMac+MacBookPro12%2C1+OSX+OSX+10.12.3+build(16D32)&version=12020010&nettype=WIFI&fontScale=100&pass_ticket=e%2FhOb%2BpI5NCO678PpRSFDDqdqcC%2Ba745Sdiwt8MM%2BgaqBZoREoBd2lGTsW3Jt5yi)里面看出实现它的是一个中国人，也是牛逼了。

其实我在学习iOS开发的时候，就知道了人家iOS用的就是约束布局，所以说Android是一直向人家靠拢嘛。好处就是，UI的层级少了，计算当然就少了，具体看以前我的blog理解这些绘制的原理即可。它也是和相对布局很类似的，默认android官方是推荐我们使用相对布局嘛，AS2.3以后就默认让我们使用约束布局了。而且使用as拖动来开发UI是非常方便的了，再结合xml布局的优势，不像ios那样的理解xib的成本，android就牛逼了。

当布局扁平化了以后，层级少了，那么性能效率肯定高了，网上也很多相关的[资料](https://mp.weixin.qq.com/s?__biz=MzIyMjQ0MTU0NA==&mid=2247483858&idx=1&sn=6d4a2151d556cc0d5587e657d01e9f72&chksm=e82c38f5df5bb1e3763ba90de29e88dba52cf7d1b1cc44745e18368cad10c2104079c6ada31b&mpshare=1&scene=1&srcid=0412mLWoLJMdYXuarTdPdGYc&key=58e96214a248c9b215123634d76bc292fd4bccec99555772d35e603f980aa8cb7be99f26e035a76278d1ec7645b6bca07c98423c8765aca39c1e06c3bf477f8701ef865df7f535fba3311917de7ef7e8&ascene=0&uin=MTMxNTU3NzYyMA%3D%3D&devicetype=iMac+MacBookPro12%2C1+OSX+OSX+10.12.3+build(16D32)&version=12020010&nettype=WIFI&fontScale=100&pass_ticket=e%2FhOb%2BpI5NCO678PpRSFDDqdqcC%2Ba745Sdiwt8MM%2BgaqBZoREoBd2lGTsW3Jt5yi)，我以后再认真学习一下。

### 2.1 配置

为了向前兼容，约束布局是在support包里面的，需要更新sdk才能使用。在SDK Tool里面的Support Repository里面会看到ConstraintLayout for Android和Solver for ConstraintLayout，都安装上就可以了。当然AS需要更新到2.3版本。

然后在gradle里面配置依赖：

>compile 'com.android.support.constraint:constraint-layout:1.0.2'

关于这个版本号去哪里找的，在sdk的目录下：

>$SDK_HOME/extras/m2repository/com/android/support/constraint

### 2.2 使用

网上太多的资料啦，我也不想搬砖，[官方文档在此](https://developer.android.com/reference/android/support/constraint/ConstraintLayout.html)，一手资料，就是英文的而已，好好学习。也有一篇[翻译](http://www.jianshu.com/p/38ee0aa654a8)，还不错。

另外，这个老外写的一篇[blog](https://codelabs.developers.google.com/codelabs/constraint-layout/index.html#0)也是还不错的，[翻译在此](http://quanqi.org/2016/05/20/code-labs-constraint-layout/)。

粗略看看上面的资料足够了，这里也不需要理解实现原理什么的，都是很简单的东西，在编辑器上面拖拽就可以了，也没有很多深奥的新概念什么的，所以看看上面资料就上手去写，遇到问题再记录在这里，这是程序员成长最快的方式。

## 3. RecyclerView

其实RecyclerView出来的时候就知道了，那个时候才做android，连listview都没用熟悉，然后又没有特别的需求，基本上listview都够用了，那时候也懒得去单独学习，基本上就是业务驱动方式去学习的。

google一下会有很多学习资料，我就不会重复了，这里只简单备忘记录一下，以防忘记的时候来看一下即可。

它并没有替换ListView，使用起来其实也挺复杂的，没有比ListView封装了很多功能来得方便，如divider，header view, footer view等等。但是如它名字所示，他做得更好的是Recycle上。另外，横向滚动，gridview，瀑布流这些都已经实现了，还有很多的动画，这些我们都不需要去找第三方库了。

以前会觉得使用方法和设计理念这些都很复杂，不好理解，其实，熟悉ListView以后觉得其实都不复杂的。

首先当然是在xml的布局文件添加一个RecyclerView，然后也是需要设置Adapter的，且看Adapter是这样编写的：

{% highlight java %}

private class MyRecyclerViewAdapter extends RecyclerView.Adapter<MyRecyclerViewHolder> {

    @Override
    public MyRecyclerViewHolder onCreateViewHolder(ViewGroup viewGroup, int viewType) {
        return null;
    }

    @Override
    public void onBindViewHolder(MyRecyclerViewHolder myRecyclerViewHolder, int position) {

    }

    @Override
    public int getItemCount() {
        return 0;
    }
}
{% endhighlight %}

跟ListView那样每次都写一个继承BaseAdapter类似，做同样的工作，区别是还用了泛型，居然是ViewHolder。看到三个需要实现的方法是:

1.getItemCount()就不用解释了，和BaseAdapter一样的。

2.onCreateViewHolder()被回调的时候通知我们去创建ViewHolder，传入了ViewGroup和viewType，可以知道它规范了ViewHolder，不用我们每次都自己去重复实现ViewHolder，之前还在github上看BaseAdapter的开源项目，现在都没有多大必要了。其中viewType就是用来实现多种view的，直接在这里就搞定了，listview还需要多一个方法。不过如果要实现多个view类型的时候，要写多个ViewHolder是在所难免的，不然在下一个方法怎么区分？

3.onBindViewHolder()就想listview的getItemView了，每一个item都回调，可以看到的是直接传入了viewholder和position，我们直接用position在datas里面获取data就好了，不用以前baseAdapter那样再多一个方法了。

再看ViewHolder的写法，并没有多大的新奇，和我平时的写法一样。

{% highlight java %}

private class MyRecyclerViewHolder extends RecyclerView.ViewHolder {

    public MyRecyclerViewHolder(View itemView) {
        super(itemView);
    }
}

{% endhighlight %}

这些都写好了，怎么使用呢？还需要一个LayoutManager的使用，非常简单，如下：

{% highlight java %}

LinearLayoutManager linearLayoutManager = new LinearLayoutManager(context);
linearLayoutManager.setOrientation(LinearLayoutManager.HORIZONTAL);
recyclerView.setLayoutManager(linearLayoutManager);
recyclerView.setAdapter(new MyRecyclerAdapter(context, datas));

{% endhighlight %}

可以看到，使用了LinearLayoutManager，还会有其他的LayoutManager，也可以我们自己去实现，更多请看[官网](https://developer.android.com/reference/android/support/v7/widget/RecyclerView.html)。还有关于动画这些东东，关于设计理念，回收机制，实现原理，这里都不记录啦。