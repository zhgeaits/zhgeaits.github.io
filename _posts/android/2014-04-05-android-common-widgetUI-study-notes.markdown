---
layout: post
title:  "Android的常用UI学习笔记&会持续更新的!"
date:   2014-04-05 11:00:03
categories: android
supertype: career
type: android
---

* TextView可以设置这样一个属性，文字上面配一个图片。
{% highlight xml %}
android:drawableTop="@drawable/default_tip"
{% endhighlight %}

如果是用代码来写，必须给drawable调setBounds
{% highlight xml %}
Drawable drawable = getResources().getDrawable(R.drawable.onlinestate_down_btn);
drawable.setBounds(0, 0, drawable.getMinimumWidth(), drawable.getMinimumHeight());
mOnlinestate.setCompoundDrawables(null, null, drawable, null);
{% endhighlight %}

* TextView可以设置文字对齐，但是排版问题解决不了。。。
{% highlight xml %}
android:gravity="right"
{% endhighlight %}

如果textview的width和height没有match_parent的话，那么设置centerVertical和centerHorizontal就会无效。

* TextView设置滚动效果  
之前排版的问题解决不了，不如就设置一行，然后滚动。  
{% highlight xml %}
android:singleLine="true”
android:ellipsize="marquee”
android:focusableInTouchMode="true”
android:focusable="true”
android:marqueeRepeatLimit="marquee_forever”
{% endhighlight %}

上面的配置使得textview只有在获得焦点以后才会滚动，要一直滚动，需要重写三个方法。  
{% highlight java %}
@Override
protected void onFocusChanged(boolean focused, int direction,
Rect previouslyFocusedRect) {
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

* TextView设置一行文字，然后超过就显示点点。。。省略号
{% highlight xml %}
android:maxEms="6" 
android:singleLine="true"
android:ellipsize="end"
{% endhighlight %}

* TextView设置下划线：  
一种方式是，字符是固定的，则然后可以再strings.xml里面设置：  
{% highlight xml %}
<string name="str_pic_login_change"><u>换一张</u></string>
{% endhighlight %}

另外一种方式是，字符是动态变化的，则然后可以再代码里面设置：  
{% highlight java %}
TextView textView = (TextView)findViewById(R.id.testView); 
textView.setText(Html.fromHtml("<u>"+"换一张"+"</u>"));
{% endhighlight %}

* TextView设置超链接:  
再strings.xml里面设置
{% highlight xml %}
<string name="str_register_agreement">注册即表示您已同意<a href="http://3g.yy.com/notice/declare.html" style="color:0091FF">使用条款和隐私政策</a></string>
{% endhighlight %}
然后布局里面不用改什么，网上说加一个属性autoLink="web",但是我加了还是不能跳转，然后网上又有说在代码里面加这一段：  
TextView agree = (TextView) view.findViewById(R.id.register_agree);  
agree.setMovementMethod(LinkMovementMethod.getInstance());  
我发现加了更加没有，连超链接的颜色都失效了。后来发现，上面代码需要，但是autoLink属性去掉即可。

* 给一个TextView设置Text后，如何马上获得这个TextView的宽度？

* **EditText设置长度**  
editInput.setFilters(new InputFilter[] {new InputFilter.LengthFilter(2048)});

**EditText的点击事件**  
setOnTouchListener()比setOnClickListener()更快获得点击事件，后者会先弹出键盘，然后有时候就失效了。

* **修改EditText的光标**  
android:textCursorDrawable="@null"//这样表示光标颜色和字体颜色一致  
如果要修改颜色这样
这样一个drawable文件color_cursor.xml
{% highlight xml %}
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android" >
    <size android:width="3dp" />
    <solid android:color="#FFFFFF"  />
</shape>
android:textCursorDrawable="@drawable/color_cursor"
{% endhighlight %}

* 给一个View动态在代码里面设置android:layout_toLeftOf这样的属性，这个在RelativeLayout里面的属性：   
{% highlight java %}
RelativeLayout.LayoutParams params = new RelativeLayout.LayoutParams(
                    ViewGroup.LayoutParams.WRAP_CONTENT,
                    ViewGroup.LayoutParams.WRAP_CONTENT);
params.addRule(RelativeLayout.RIGHT_OF, R.id.sign_label);
params.addRule(RelativeLayout.CENTER_VERTICAL);
params.rightMargin = dip2px(this, 30);
view.setLayoutParams(params);
{% endhighlight %}

**Intent**  
启动一个activity的时候，给intent加flag可以处理activity栈：  
FLAG_ACTIVITY_CLEAR_TOP  如果新的activity实例已经在返回栈中运行，这时候会把它置顶，并且清除上面的activity。  
FLAG_ACTIVITY_SINGLE_TOP  如果返回栈中最上面的activity是正在启动的activity，那么这个实例会置于前台，不会创建新的activity。

* **判断当前的activity是否是栈顶的活动activity**  
需要添加android.permission.GET_TASKS权限，然后获得ActivityManager，再获取getRunningTasks，最后比较getClassName就可以。具体看代码：CommonUtils  
另外一种方式，可以用一个boolean变量，在onResume方法设置为true，在onPause方法设置为false，这样也可以判断。

* **使用Dialog的一个报错**  
一般来讲，如果activity要关闭的时候show一个dialog，就会报这个错。但是我在这个大项目里面，明明是没有关闭的，也报错，估计是回调那些太复杂了导致的，具体原因我也没有查出来。。。。  
Unable to add window  token android.os.BinderProxy is not valid; is your activity running?  
解决方法就是:  
if(!isFinishing()) {  
   // show dialog  
}

**progressbar的左右宽度**  
progressbar左右的padding好像默认不是0，所以要设置一下  
android:paddingLeft="0px"  
android:paddingRight="0px"  

{% highlight xml %}
<ProgressBar
    android:id="@+id/progress_horizontal"
    style="?android:attr/progressBarStyleHorizontal"
    android:progressDrawable="@drawable/capture_progress_style"
    android:layout_width="match_parent"
    android:layout_height="6dp"
    android:max="100"
    android:progress="10"
    android:secondaryProgress="20"
    />

//其中capture_progress_style.xml配置如下：

<layer-list xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:id="@android:id/background">
        <shape>
            <corners android:radius="1.0dip"/>
            <gradient
                    android:startColor="#1Effffff"
                    android:endColor="#1Effffff"
                    android:angle="270.0"/>
        </shape>
    </item>
    <item android:id="@android:id/progress">
        <clip>
            <shape>
                <corners android:radius="1.0dip"/>
                <gradient
                        android:startColor="#ffa401"
                        android:endColor="#ffa401"
                        android:centerColor="#ffa401"
                        android:angle="270.0"/>
            </shape>
        </clip>
    </item>
    <item android:id="@android:id/secondaryProgress">
        <clip>
            <shape>
                <corners android:radius="1.0dip"/>
                <gradient
                        android:startColor="#89ffa401"
                        android:endColor="#89ffa401"
                        android:centerColor="#89ffa401"
                        android:angle="270.0"/>
            </shape>
        </clip>
    </item>
</layer-list>

{% endhighlight %}

**ActionBar**  
是3.0以后才有ActionBar，可以在menu/main.xml配置bar上面的item，然后在Activity的onCreateOptionsMenu里面创建。

**Application**  
公司的项目里面，是自己继承了Application的，当出现异常崩溃以后，发现App的oncreate执行了一次，虽然不知道具体是什么，猜测是有一些服务例如是sticky的导致系统要去启动Application，但是不会启动应用。

**include和viewstub**  
两者都是可以直接引入一个layout，达到布局重用。区别是，前者是随着引用以后跟着父布局即时渲染，这样效率不高；后者是你去调用viewstub.inflate()方法以后才会去渲染。

**如果想要在代码中设置 android:layout_centerInParent属性，则可以在代码中这样写：**  
layoutParams.addRule(RelativeLayout.CENTER_IN_PARENT);

**Activity屏幕旋转会再次执行oncreate**   
给activity配置属性android:configChanges="orientation|screenSize"，告诉activity不要去执行oncreate，我们自己处理，然后就会收到回调：  
{% highlight java %}
public void onConfigurationChanged（Configuration newConfig） {
       
		super.onConfigurationChanged（newConfig）;
        if （newConfig.orientation==Configuration.ORIENTATION_LANDSCAPE） {
            
        } else {
		
        }
}
{% endhighlight %}

* dp和pix之间的转换：  
{% highlight java %}
public static int dip2px(Context context, float dpValue) {
	final float scale = context.getResources().getDisplayMetrics().density;
	return (int) (dpValue * scale + 0.5f);
}

public static int px2dip(Context context, float pxValue) {
	final float scale = context.getResources().getDisplayMetrics().density;
	return (int) (pxValue / scale + 0.5f);
}
{% endhighlight %}

获取view在屏幕的坐标：  
{% highlight java %}
int[] location = new int[2];
view.getLocationOnScreen(location);
int x = location[0];
int y = location[1];
{% endhighlight %}

**dialog的edittext无法弹出键盘问题**  
//只用下面这一行弹出对话框时需要点击输入框才能弹出软键盘  
window.clearFlags(WindowManager.LayoutParams.FLAG_ALT_FOCUSABLE_IM);  
//加上下面这一行弹出对话框时软键盘随之弹出  
window.setSoftInputMode(WindowManager.LayoutParams.SOFT_INPUT_STATE_ALWAYS_VISIBLE);  
