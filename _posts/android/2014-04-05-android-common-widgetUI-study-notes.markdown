---
layout: post
title:  "Android的UI使用笔记"
date:   2014-04-05 11:00:03
categories: android
supertype: career
type: android
---

>这里只记录一些常用的UI View的使用技巧，而非原理性东西。虽然网上一搜都有，但是也省得每次都找得这么麻烦，于是都记录在一个blog里面，找起来也方便，遇到过的坑记录也容易找到。

## 0. View

#### include和ViewStub

两者都是可以直接引入一个layout，达到布局重用。区别是，前者是随着引用以后跟着父布局即时渲染，这样效率不高；后者是你去调用viewstub.inflate()方法以后才会去渲染。

#### 获取view在屏幕的坐标

{% highlight java %}

int[] location = new int[2];
view.getLocationOnScreen(location);
int x = location[0];
int y = location[1];

{% endhighlight %}

## 1. TextView

#### TextView可以设置这样一个属性，文字上面配一个图片。

{% highlight java %}

//布局里面设置
android:drawableTop="@drawable/default_tip"

//通过代码来设置，必须给drawable调setBounds
Drawable drawable = getResources().getDrawable(R.drawable.onlinestate_down_btn);
drawable.setBounds(0, 0, drawable.getMinimumWidth(), drawable.getMinimumHeight());
mOnlinestate.setCompoundDrawables(null, null, drawable, null);

{% endhighlight %}

#### TextView可以设置文字对齐，但是排版问题解决不了。

如果textview的width和height没有match_parent的话，那么设置centerVertical和centerHorizontal就会无效。

{% highlight xml %}

android:gravity="right"

{% endhighlight %}

#### TextView设置滚动效果

之前排版的问题解决不了，不如就设置一行，然后滚动。

{% highlight xml %}

android:singleLine="true”
android:ellipsize="marquee”
android:focusableInTouchMode="true”
android:focusable="true”
android:marqueeRepeatLimit="marquee_forever”

{% endhighlight %}

上面的配置使得textview只有在获得焦点以后才会滚动，如需要一直滚动，需要重写三个方法。

{% highlight java %}

@Override
protected void onFocusChanged(boolean focused, int direction, Rect previouslyFocusedRect) {
    // TODO Auto-generated method stub
    if(focused)
       super.onFocusChanged(focused, direction, previouslyFocusedRect);
}
@Override
public void onWindowFocusChanged(boolean hasWindowFocus) {
    // TODO Auto-generated method stub
    if(hasWindowFocus)
        super.onWindowFocusChanged(hasWindowFocus);
}
@Override
public boolean isFocused() {
    return true;
}

{% endhighlight %}

#### TextView设置一行文字，然后超过就显示点点...省略号

{% highlight xml %}

android:maxEms="6" 
android:singleLine="true"
android:ellipsize="end"

{% endhighlight %}

#### TextView设置下划线

一种方式是，字符是固定的，则然后可以再strings.xml里面设置：  

{% highlight xml %}

<string name="str_pic_login_change"><u>换一张</u></string>

{% endhighlight %}

另外一种方式是，字符是动态变化的，则然后可以再代码里面设置：

{% highlight java %}

textView.setText(Html.fromHtml("<u>"+"换一张"+"</u>"));

{% endhighlight %}

#### TextView设置超链接

在strings.xml里面设置

{% highlight xml %}

<string name="str_register_agreement">注册即表示您已同意
<a href="http://3g.yy.com/notice/declare.html" style="color:0091FF">使用条款和隐私政策</a>
</string>

{% endhighlight %}

然后布局里面不用改什么，网上说加一个属性`autoLink="web"`，但是我加了还是不能跳转，然后网上又有说在代码里面加这一段

>textView.setMovementMethod(LinkMovementMethod.getInstance());

我发现加了更加没有，连超链接的颜色都失效了。后来发现，上面代码需要，但是autoLink属性去掉即可。

## 2. EditText

#### EditText设置长度

>editInput.setFilters(new InputFilter[] {new InputFilter.LengthFilter(2048)});

#### EditText的点击事件

setOnTouchListener()比setOnClickListener()更快获得点击事件，后者会先弹出键盘，然后有时候就失效了。

#### 修改EditText的光标

>android:textCursorDrawable="@null"//这样表示光标颜色和字体颜色一致

如果要修改颜色就这样先定义一个drawable的xml文件color_cursor.xml

{% highlight xml %}

<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android" >
    <size android:width="3dp" />
    <solid android:color="#FFFFFF"  />
</shape>

android:textCursorDrawable="@drawable/color_cursor"

{% endhighlight %}

#### Dialog的Edittext无法弹出键盘问题

{% highlight java %}

//只用下面这一行弹出对话框时需要点击输入框才能弹出软键盘
window.clearFlags(WindowManager.LayoutParams.FLAG_ALT_FOCUSABLE_IM);
//加上下面这一行弹出对话框时软键盘随之弹出
window.setSoftInputMode(WindowManager.LayoutParams.SOFT_INPUT_STATE_ALWAYS_VISIBLE);

{% endhighlight %}

## 3. LayoutParams

#### 给一个View动态在代码里面设置像android:layout_toLeftOf这样的属性

{% highlight java %}

RelativeLayout.LayoutParams params = new RelativeLayout.LayoutParams(
                    ViewGroup.LayoutParams.WRAP_CONTENT,
                    ViewGroup.LayoutParams.WRAP_CONTENT);
params.addRule(RelativeLayout.RIGHT_OF, R.id.sign_label);
params.addRule(RelativeLayout.CENTER_VERTICAL);
params.addRule(RelativeLayout.CENTER_IN_PARENT);
params.rightMargin = dip2px(this, 30);

view.setLayoutParams(params);

{% endhighlight %}

## 4. ProgressBar

#### ProgressBar的左右宽度

progressbar左右的padding好像默认不是0，所以要设置一下

{% highlight xml %}

android:paddingLeft="0px"
android:paddingRight="0px"

{% endhighlight %}

## 5. Switch

这个控件是4.0以后才有的，如果想要低版本用，可以直接拿官网的源码来用。。。。这也说明了一点，其实如果有时候一些新版本的控件也可以这样拿来用。

## 6. ListView

#### listview 每个item的高度

发现如果listview的每一项item没有图片的话，不管怎么设置item的布局高度都是无效的，网上查过以后，发现在item的布局里面设置最小高度就可以了。

{% highlight xml %}

android:minHeight="50dp"

{% endhighlight %}

#### listview/viewpager去掉边缘滑动阴影和按下颜色（这些效果耗内存）

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

- 默认时的背景图片：什么都不设置
- 没有焦点时的背景图片：android:state_window_focused="false"
- 非触摸模式下获得焦点并单击时的背景图片：android:state_focused="true" android:state_pressed="true"
- 触摸模式下单击时的背景图片：android:state_focused="false" android:state_pressed="true"
- 选中时的图片背景：android:state_selected="true"
- 获得焦点时的图片背景：android:state_focused="true"

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

#### 滚动

listview作为聊天界面的时候，要自动滚到底部，可以设置一个定时器，然后执行下面代码就可以了。

>chatList.setSelection(fragmentAdapter.getCount() - 1);

setSelection是直接跳过去，而smoothScrollToPosition是滚过去。

遇到一个奇怪问题，在聊天界面，聊天内容如果用的是listview的话，点击输入框弹出键盘以后，listview会自动滚到底部，但是公司的是用PullToRefreshView，虽然是继承listview的，但是就不会滚动到底部，试过用上面的定时器方法，但是时间不好控制。也试过重写Layout来捕获OnResize方法，但是竟然还是无效。后来在stackoverflow上面找到了，加入这个属性就行：

>android:transcriptMode="alwaysScroll"

listview默认应该是normal，估计PullToRefreshView做了手脚。

#### 分割线

{% highlight java %}

android:divider="@null";
android:divider="@drawable/listview_horizon_line"

{% endhighlight %}

#### 每项的间距

android:dividerHeight="10dp"

#### StickyListHeaders

给listview设置一个黏住，浮起来不动的标题：[StickyListHeaders](https://github.com/emilsjolander/StickyListHeaders)

#### 根据每一项的高度来动态设置listview的高度

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

#### getView()和notifyDataSetChanged()

一般调用notifyDataSetChanged()就会触发getView()方法。然后我在项目这里发现了不管怎么调用notifyDataSetChanged都不会触发getView，想了一天都不能解决，后来才发现，必须在UI线程调用啊。

## 7. ViewPager

viewpager可以实现屏幕滑动的效果，用来做画廊，启动界面这些都不错，而且比Flipper，Gallery都快，顺畅。可以考虑把tongli的那个图片换成viewpager。

#### 官方的viewpager使用很简单

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

#### 设置tab

一般结合viewpager使用的时候在顶部有一些tab，当滑动viewpager的时候，tab也跟着变化。官方的有这两个控件，他们都是作为viewpager的子控件使用：

PagerTabStrip: 交互式  
PagerTitleStrip: 非交互式  

PagerTabStrip：  
1.点击上面的标题可以实现ViewPager的切换。  
2.选中的文字下方包含指引线  
3.显示全宽下划线（setDrawFullUnderline）

PagerTitleStrip：  
1.点击上面的标题无反应。  
2.无上述描述。

代码上没多少区别，viewpager和strip无需再绑定了，只要在上面的adapter里面再重写一个方法和设置title的data

{% highlight java %}

@Override  
public CharSequence getPageTitle(int position)   {  
     return Titles.get(position);  
}

{% endhighlight %}

另外，strip有很多属性可以设置的。

有一个开源的strip，那些tap是可以滑动的：[pagerslidingtabstrip](http://blog.csdn.net/top_code/article/details/17438027)，如果要用它的时候再认真去学，公司就是用这个的，然后再适当修改为自己需求的。

比较成熟和出名的是这个[Android-ViewPagerIndicator](http://viewpagerindicator.com/)

#### FragmentPagerAdapter

viewpager的adapter的另外一种写法，不是继承PagerAdapter，而是继承FragmentPagerAdapter，没什么难的，只是viewpager的每一个view都是fragment而已。而且，这个adapter还需要传FragmentManager进去。

#### FragmentStatePagerAdapter

FragmentPagerAdapter更多的用于少量界面的ViewPager，比如Tab。划过的fragment会保存在内存中，尽管已经划过。而FragmentStatePagerAdapter和ListView有点类似，会保存当前界面，以及下一个界面和上一个界面（如果有），最多保存3个，其他会被销毁掉。

#### FixedFragmentStatePagerAdapter

可以google一下这个，说是修复被回收后重启时不能从fragmentstatepageradapter恢复的bug。但是我还是不太懂，先记录下来，也许以后遇到这个问题就懂了。

#### setOffscreenPageLimit

设置`mPager.setOffscreenPageLimit(3)`viewpager的适配器会预装在3个view，包含当前显示的view，一共4个，这是一个窗口，移动窗口时候，后面加载一个，前面销毁一个，最好能重写adapter的onDestoryItem方法。如果fragment缓存了的话，就不会去执行getItem方法，就会不去new Fragment了。

#### 设置自动滑动的切换速度

由于公司项目有需求，viewpager加一个按钮，点击然后滑动到下一个页面，直接去调用`mViewPager.setCurrentItem(mPos, true)`发现切换得很快。调查发现，可以用反射修改viewpager的scroller属性。

{% highlight java %}

Field sField = ViewPager.class.getDeclaredField("mScroller");  
sField.setAccessible(true);  
mScroller = new FixedSpeedScroller(mViewPager.getContext(), mDuration);  
sField.set(mViewPager, mScroller);  

{% endhighlight %}

给viewpager设置onTouch事件，如果返回true则可以屏蔽viewpager手动滑动。

#### 滑动页面时对按钮进行渐变效果

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

#### 要拿到viewpager的指定或者当前子view方法

方法一：创建子view的时候设置一个ID，然后可以根据这个id来find。

{% highlight java %}

view.setId(index);
mPager.findViewById(position);

{% endhighlight %}  

方法二：在adapter里面设置。

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

## 8. Spannable

在聊天的应用里面这个非常有用，输入框显示图片，聊天内容显示表情，图片，语音等等。  

SpannableString实现了CharSequence，Spannable接口；SpannableStringBuilder实现CharSequence，Spannable，Editable接口；Editable继承CharSequence，Spannable接口；ImageSpan继承DynamicDrawableSpan。

EditText里面放的一般是SpannableStringBuilder。new一个ImageSpan，然后把图片或者表情传进去。然后拿到EditText里面的Editable对象后调用spannable.setSpan(what, start, end, flag);what传的就是ImageSpan对象，start, end就是替换edittext里面的字符串。

flag参数有下面这些，它是用来标识在 Span 范围内的文本前后输入新的字符时是否把它们也应用这个效果：

Spanned.SPAN_EXCLUSIVE_EXCLUSIVE(前后都不包括)  
Spanned.SPAN_INCLUSIVE_EXCLUSIVE(前面包括，后面不包括)  
Spanned.SPAN_EXCLUSIVE_INCLUSIVE(前面不包括，后面包括)  
Spanned.SPAN_INCLUSIVE_INCLUSIVE(前后都包括)  

除了ImageSpan外，还有StyleSpan，URLSpan等等很多。

#### 给TextView高亮关键字

{% highlight java %}

SpannableString span = new SpannableString(title);  
span.setSpan(new ForegroundColorSpan(getResources().getColor(R.color.top_bar_line_color)), start, end, Spannable.SPAN_EXCLUSIVE_EXCLUSIVE);  
title.setText(span);

{% endhighlight %}

如此的简单！~之前公司的项目，一个搜索功能，以前的人写的时候，在搜索结果页要把关键字高亮，然他就用String.replaceAll()方法来把关键字替换html的<span>标签。然后没有处理到一些情况，如果用户输入正则表达式就会崩溃。因为这个方法就是用正则表达式来替换的。我开始用Matcher.quoteReplacement(searchKey)，但是会抛OOM错，由于转移递归了。后面处理方法就是遍历字符串，用spannable高亮了。

## 9. Canvas

Canvas是画布的意思，没有深入了解过这个类，不知道是不是一块内存，不过，可以理解为一块显示区域，因为，画布嘛，不就是给人看的么，内存又不是给人看的，所以，bitmap才是一块内存，包含像素，要把bitmap画到canvas上面去，当然还可以画很多元素。一般，在View里面的onDraw方法都带有一个canvas，可以用来画东西。调用invalidate()方法会触发onDraw()方法。

#### 更多canvas的方法：

>public void drawArc (RectF oval, float startAngle, float sweepAngle, boolean useCenter, Paint paint);  

oval是圆所在的矩阵，startAngle是起始角度，sweepAngle是弧的角度，useCenter是否显示半径连线，即是否画到圆心，最后是paint画笔。

>public void drawBitmap(Bitmap bitmap, float left, float top, Paint paint);

画bitmap，left，top都容易理解，就是距离左边和上边的距离。

>public void drawBitmap(Bitmap bitmap, Rect src, Rect dst, Paint paint);

画bitmap，src是要截取原本bitmap的哪些区域，传null进去就是画整个bitmap；dst是要画在屏幕的哪些区域，Rect(left, top, right, bottom)，left和top是左上角距左边和上边的距离，right是右上角（右边那条边）距左部的距离，bottom是左下角（底边）距顶部的距离。

>public void drawRoundRect(@NonNull RectF rect, float rx, float ry, @NonNull Paint paint);

画圆角矩阵，rect就是这个矩阵的坐标，paint是画笔，rx是x方向上的圆的半径，ry是y方向上的圆的半径，比较难理解，取其中一只角，可以平均切成两端弧，一段是水平方向（x方向），一段是垂直方向（y方向），这两段弧分别是来自不同半径的圆，这样就理解了。

>public void drawText(@NonNull String text, float x, float y, @NonNull Paint paint);

画文字，x和y分别是文字的起始位置，是相对左上角的，值得注意的是，文字是从baseline那里开始画起的，并不是左上角。

#### Paint

Paint是画笔的意思，其实画笔就是带有颜色(Color)和样式(Styles)这些属性。调用canvas画东西的时候，必须传入一个画笔。画笔还可以设置锯齿，argb，字体大小，边框等属性。

更多Paint的方法：  

paint.setAntiAlias(true);//消除锯齿  
paint.ascent();  
paint.descent();//这些方法获取一定的距离值,如下图

![alt Baseline](/image/baseline.jpg "Baseline")  

1.基准点是baseline  
2.ascent：是baseline之上至字符最高处的距离  
3.descent：是baseline之下至字符最低处的距离  
4.leading：是上一行字符的descent到下一行的ascent之间的距离，也就是相邻行间的空白距离  
5.top：是指的是最高字符到baseline的值，即ascent的最大值  
6.bottom：是指最低字符到baseline的值，即descent的最大值  

#### Color

Color就是颜色类，里面定义不少常用颜色的常量值，也有不少好用的方法，例如分别可以提取argb的值。

#### ColorMatrix

ColorMatrix是颜色矩阵，非常厉害的类，理解可能比较困难，可以参考[官网](http://developer.android.com/reference/android/graphics/ColorMatrix.html)。可以通过矩阵相乘来改变颜色值，这个矩阵是4x5的，乘以颜色向量[R,G,B,A]，矩阵乘法还记得吧，所以修改矩阵就可以修改argb了。可以参考我的RBPlayer里面的colorview类。这个的类的应用一般用在图片美化方面，改变图片的样式，泛黄啊，变灰啊什么的。

#### Matrix

Matrix是修改位置，例如进行旋转，平移什么的，也非常强大，目前还没用到过，但是有一些比较厉害的应用就是实现Folding Layout，可以去Google一下。

#### 其他

对于自定义绘制圆形，或者圆角图片，可以使用Xfermode，Shader。