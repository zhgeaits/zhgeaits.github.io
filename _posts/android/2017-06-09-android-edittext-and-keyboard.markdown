---
layout: post
title:  "Android的输入框EditText和键盘的相爱相杀血案"
date:   2017-06-09 11:00:03
categories: android
supertype: career
type: android
---

>还记得大四的时候转做Android开发，买了这本书来学习UI，[精彩绝伦的Android UI设计：响应式用户界面与设计模式 [Smashing Android UI]](https://item.jd.com/11332543.html)，初接触到输入框弹出键盘，分了两种模式`adjustPan`和`adjustResize`，当时也只是学习，理解也只是从界面上感知，并没有去实际开发业务需求，所以不会有很多的探索。关于书的源码在此[GitHub开源](https://github.com/JuhaniLehtimaeki/smashing_android_ui_example_app)，居然也是我第一个star的项目，不禁想到自己入门多晚啊！

毕业以后在YY，接触太多的输入框EditText了，踩过很多坑，那时候并没有记录多少下来，关于Spannable啊，关于监听文字长度啊，中午长度处理啊，自定义表情长度啊，Emoji表情长度啊，关于兼容特殊手机如何获取焦点然后弹出键盘啊，等等，坑是都踩过了一遍，可惜并没有多少记录下来。

今天在公司做一个需求的时候，再次掉入这个坑里面了，原本以为简单Google一下就能解决的，没想到在这里花了不少时间，去年同事也遇到这个问题，不过没有一个比较好的解决方法。于是想想，都认真总结了一下，还是记录一下吧。

## 1.Requirements

需求很简单，如下面的界面所示，Activity处于全屏的状态，头部UI是一个返回的icon和一个功能icon，底下是一个全屏的SurfaceView在播放视频，Bottom下面是一个EditText输入框和一个按钮，如下图所示：

![alt edittext](/image/edittext_keyboard_1.png "edittext")

然后点击输入框以后弹出键盘如下所示：

![alt keyboard](/image/edittext_keyboard_2.png "keyboard")

看似简单，那么问题来了，不管我在AndroidManifest.xml的Activity那里怎么设置`android:windowSoftInputMode="adjustPan"`还是`adjustResize`，都是达不到预期效果，出现各种奇怪的现象，要么就是键盘覆盖在上面，要么就是输入框不知道飞到哪里去了！~~~

于是开始各种Google，出来的各种解决方案都似乎没什么用，一来，网上的文章都比较旧了，二呢，他们总结出来的方法并没有讲清楚问题本质和解决方法的结果，也就是文章写得太烂了，都是搬砖copy过来的。比如说采用根View是ScrollView这个就没讲清楚，基本上没啥用，也需要读者自行测试；另外当然自定义Layout可以解决，但是并不方便。大部分的文章”讲故事“能力太弱了，好不容易找到一篇跟[WebView相关](https://www.diycode.cc/topics/383)的，讲得挺好的，却不能解决我的问题，罢了罢了，也算打开了我的思路。

## 2.About the Fucking windowSoftInputMode

那就先来简单回顾一下windowSoftInputMode是个什么鬼吧，既然它是Activity的属性，那就Google一下咯，找到[官网的描述](https://developer.android.com/guide/topics/manifest/activity-element.html#wsoft)，好吧，就不搬砖了，主要分`stateXXX`和`adjustXXX`，前者是显示状态，后者是调整布局。

* adjustPan：不调整Activity主窗口的尺寸来为软键盘腾出空间，而是自动平移窗口的内容，使当前焦点永远不被键盘遮盖，让用户始终都能看到其输入的内容。这通常的状况看起来就是输入框浮在了键盘上面。

* adjustResize：始终调整Activity主窗口的尺寸来为屏幕上的软键盘腾出空间。效果就是把Activity的布局压小了，显示在键盘上面。所以说是会回调到布局的`onSizeChange()`方法的

**关于adjustPan是怎么实现的，到底是改变了哪个布局的大小？是不是单独把输入框提出来了？这个没有深入研究源码。以后有兴趣再深入学习**

另外，关于这个属性定义在代码这里`WindowManager.softInputMode`、`WindowManager.SOFT_INPUT_ADJUST_RESIZE`和`WindowManager.SOFT_INPUT_ADJUST_PAN`，这里的注释也有很好的说明。

## 3.Write the Shit Test Code

那好吧，其实，Android真的很多坑，我们又还需要兼容旧机器，还有更多奇葩的问题了，你又不可能一劳永逸的解决一类问题，比如说这次的问题，我就只能针对我这次的业务场景解决我的问题了，接下来你将会看到恶心到让你把上个月吃过的都呕吐出来的坑-:)

下面写了一个[Demo](http://zhgeaits.me/EditTextHateKeyboard.zip)，将会演示下面所有的实现。

写一个类似的界面，如下:

![alt keyboard](/image/edittext_keyboard_3.png "keyboard")

### 3.1 测试全屏和非全屏下的windowSoftInputMode

#### 3.1.1 测试一：非全屏下，使用`adjustPan`模式

这个case下，点击输入框以后，输入框能会浮在键盘上，底下Button是会被键盘覆盖，显然Activity是不会不改变大小的（不需要验证，官方说明就是这个意思，当然可以自行设置一张图片观察图片是否变形等，最合理合适监听onSizeChange方法是否有回调），但是会上移动，头部的UI看不见了。收起键盘以后，一切显示正常，多次操作均正常显示。效果如图所示：

![alt keyboard](/image/edittext_keyboard_4.png "keyboard")

#### 3.1.2 测试二：非全屏下，使用`adjustResize`模式

这个case下，点击输入框以后，**输入框不会浮在键盘上**，可能看到底下的Button，显然Activity整体大小改变，头部的UI还能看得见。收起键盘以后，一切显示正常，多次操作均正常显示。效果如图所示：

![alt keyboard](/image/edittext_keyboard_5.png "keyboard")

#### 3.1.3 测试三：全屏下，使用`adjustPan`模式

这个case下也是正常的，效果和测试一是一样的，如图所示：

![alt keyboard](/image/edittext_keyboard_6.png "keyboard")

#### 3.1.4 测试四：全屏下，使用`adjustResize`模式

**这个case居然失效了**，理论上应该改变Activity的大小，可以看到底下的Button，但是，效果居然和测试三一样了。收起键盘，多次点击都如此。也就是说adjustResize变成了adjustPan。效果如图所示：

![alt keyboard](/image/edittext_keyboard_6.png "keyboard")

#### 3.1.5 小结对比

以上四个测试case简单总结了`adjustPan`和`adjustResize`两种模式的使用对比，并且区分了全屏和非全屏的情况之下，发现在全屏下，adjustResize会失效，对比如下：

<table>

<tr>
<th></th><th>adjustPan</th><th>adjustResize</th>
</tr>

<tr>
<td>非全屏</td><td>正常</td><td>正常</td>
</tr>

<tr>
<td>全屏</td><td>正常</td><td style="color: #ea3f40;">失效变为adjustPan的效果</td>
</tr>

</table>

### 3.2 测试全屏和非全屏下的加入SurfaceView的windowSoftInputMode

#### 3.2.1 测试五：非全屏下，使用`adjustPan`模式

来了，来了，第一波呕吐来了！这个case真挺奇葩的！其实相当于在测试一加了一个SurfaceView，测试也挺正常的，效果图和测试一是一样，差别在于背景是SurfaceView以后就是全黑的了，就不放图了。但是！但是！但是！我把Activity换了一个主题，这个主题如下：

`<style name="AppTheme2" parent="@android:style/Theme.NoTitleBar">`

原本的主题是：

`<style name="AppTheme" parent="Theme.AppCompat.NoActionBar">`

那么奇葩的情况就出现了！首先主题变了，输入框的样式也变了，然后点击输入框以后很正常的把输入框浮在输入框上面，结果收起键盘的时候，头部的UI不见了！~不见了！~再也收不回来了！~尼玛，多次点击也没用了，输入框能一直显示在键盘之上！如图所示：

![alt keyboard](/image/edittext_keyboard_7.png "keyboard")

实际上，底下布局也就是SurfaceView肯定是被上移了的，理论上来说，adjustPan是不应该改变底下布局的。你可以用SurfaceView播放视频测试一下就发现了。

<p style="color: #ea3f40;">另外，3.1里面的测试，使用同样的这个主题测试，效果是一样的，并没有特别的奇葩现象。再次说明，Android旧版本是很多问题的，使用AppCompat的主题就修复了。</p>

#### 3.2.2 测试六：非全屏下，使用`adjustResize`模式

这个case也是还正常的，效果和测试二一样，使用测试二的效果图如下，区别只是背景全黑了。采用上面的奇葩主题测试也是正常的。**不过就是地下的SurfaceView是没有被压缩的，也没有被上移。**

![alt keyboard](/image/edittext_keyboard_5.png "keyboard")

#### 3.2.3 测试七：全屏下，使用`adjustPan`模式

这个case不正常了，多次点击输入框以后都能浮在键盘之上，但是收起键盘以后头部UI不见了，和**测试五**采用奇葩主题一样了，效果如下。**另外采用了奇葩主题测试，同样的异常，通常appCompat的主题都异常了，奇葩主题肯定异常。**

![alt keyboard](/image/edittext_keyboard_8.png "keyboard")

#### 3.2.4 测试八：全屏下，使用`adjustResize`模式

这个case和**测试七**一样的效果，adjustResize变成了adjustPan，输入框是一直能浮在键盘之上，并且头部UI收不回来了，不见了！效果图就不放了。

#### 3.2.5 小结对比

以上四个测试只是增加了SurfaceView就出现了完全不一样的效果了，真是一个大坑啊，根据网上别人的经验，估计放WebView也是会变得奇葩的。

<table>

<tr>
<th></th><th>adjustPan</th><th>adjustResize</th>
</tr>

<tr>
<td>非全屏</td><td style="color: #ea3f40;">AppCompat主题正常，非AppCompat主题异常，不见了头部UI，输入框一直能看见。（SurfaceView上移）</td><td>正常，SurfaceView不被压缩。</td>
</tr>

<tr>
<td>全屏</td><td style="color: #ea3f40;">异常，头部UI不见了，输入框一直能看见。（SurfaceView上移）</td><td style="color: #ea3f40;">异常失效变为adjustPan的效果，并且头部UI不见了，输入框一直能看见。（SurfaceView上移）</td>
</tr>

</table>

### 3.3 令人呕吐的变态属性

在上面的八个测试中，3.1的四个基本上都是可以算是正常的，就算在**全屏下使用adjustResize失效变为adjustPan的效果**，也是可以接受使用的。而3.2中的四个，在非全屏下也还是能使用的。

不过，demo的EditText都是很简单的，通常我们都会设置一些自定义属性，而我就是在设置这些自定义属性的时候发现了居然他妈的不一样的现象出现了！~

#### 3.3.1 神奇组合：inputType="text" + gravity="center"，或者singleLine="true" + gravity="center"

分别对3.1的四个测试中的EditText增加这两对组合属性，于是乎出现了奇葩的现象，效果图就不需要上了，总结对比如下：

<table>

<tr>
<th></th><th>adjustPan</th><th>adjustResize</th>
</tr>

<tr>
<td>非全屏</td><td style="color: #ea3f40;">第一次正常，后面被键盘覆盖</td><td>正常</td>
</tr>

<tr>
<td>全屏</td><td style="color: #ea3f40;">完全被键盘覆盖</td><td style="color: #ea3f40;">完全被键盘覆盖</td>
</tr>

</table>

不管什么inputType都是一样，而gravity只有center或者center_horizontal才会奇葩。**另外，有时候测试全屏的时候，第一次是正常的，后面就被键盘覆盖了**，这个跟项目环境版本这些都有关系，不过意义已经不大了，反正都是不正常使用。

可以看到这两个组合都是有`gravity=”center“`属性的，似乎它是关键，不过单独试着这个属性又是正常的，只有组合情况才会异常，而且三个一起组合，也是少不了gravity才会异常。

同样分别对3.2中的四个测试增加这两对属性进行测试。也是不需要上效果图了，可以自行跑demo，总结对比如下：

<table>

<tr>
<th></th><th>adjustPan</th><th>adjustResize</th>
</tr>

<tr>
<td>非全屏</td><td style="color: #ea3f40;">第一次正常，后面被键盘覆盖。使用非compat主题后，第一次弹出键盘正常，收起键盘不见了头部UI，第二次键盘覆盖输入框，出现头部UI</td><td>正常</td>
</tr>

<tr>
<td>全屏</td><td style="color: #ea3f40;">完全被键盘覆盖</td><td style="color: #ea3f40;">完全被键盘覆盖</td>
</tr>

</table>

可以看到非全屏的时候，使用非AppCompat主题后很奇葩，在全屏的时候测试也会出现这样的情况，还要看环境，反正说不准，结果都是异常了。

#### 3.3.2 小结

目前我只发现了这三个属性居然如此大的影响原本的效果，而且这三个属性是很容易被我们设置的，基本上只要使用了这三个属性，那么整个输入框就无法使用了，当然，有一个例外，就是无论什么情况，非全屏下使用adjustResize都是正常的！所以，我们也看到很多app都是这样处理的。

这在demo里面已经算是没有出现一些变态的情况了，我在其他项目测试的时候，上面全屏的四个case，在第一次都是正常的，收起键盘以后，有时候头部UI会不见了，第二次就异常了，输入框被键盘覆盖掉，头部UI再次出现。

如果都像demo那样不变态，如果都能接受全屏以后使用adjustResize的方式，那么可以用网上的一个方法[AndroidBug5497Workaround](https://stackoverflow.com/questions/7417123/android-how-to-adjust-layout-in-full-screen-mode-when-softkeyboard-is-visible/19494006#19494006)，实际上是改变布局大小。

<p style="color: #ea3f40;">然而还有更多的属性在使用的时候不知道会出现什么情况，通常都是会导致输入框被键盘盖住，或者压缩界面等等，adjustPan也会失效，反正业务场景复杂的情况多的是。</p>

使用AndroidBug5497Workaround也不好解决！比如说，上面非全屏adjustPan，不管有没有SurfaceView的时候，也就是只要出现第一次正常，第二次就会被键盘覆盖的情况，你使用AndroidBug5497Workaround，就会出现如下奇葩的现象，修改布局大小完全失效了，第二次才会正常。我也试过debug调试AndroidBug5497Workaround类，设置高度是完全正确的，但是就是奇葩的异常了。

![alt keyboard](/image/edittext_keyboard_9.png "keyboard")

这个图只是展示了非全屏的情况，有时候全屏的情况也是会出现的，只要是第一次正常，第二次以后异常的case使用AndroidBug5497Workaround都会这样。具体原因我也无法解释。

## 4.新的思路

从第三节的测试看到，似乎根本没有办法解决我目前的需求，我想要的是adjustPan的效果，自定义修改布局大小的思路根本不行。也知道了，异常的奇葩的变态的情况随时会出现，根本无法捉摸，也无法控制。

由之前对[View的学习](http://zhgeaits.me/android/2014/07/27/android-ui-view-study-notes.html)知道，其实，Activity底下是由Window控制View的，键盘对View产生影响是作用到Window上，而非Activity，也就是说，只要新建立一个Window就可以有新的一个View体系了，进而去掉SurfaceView了，不受它的影响。而Dialog不就是这个最好的实现吗？

### 4.1 Using the Fucking Dialog

首先，从上面的16个测试可以得出了各种显示现在结论，就算换到Dialog的Window也是一样的效果，因为根本原因是一致的，这点我就不测试了，如果不相信，还是可以自行测试的。

目前可以知道：

* adjustResize的模式，所有情况下，**只要非全屏都是正常的。**
* adjustResize的模式，全屏情况下，正常情况是退化为adjustPan模式，异常情况是完全不能使用，被键盘覆盖。

由最初的需求可以知道，我们是想在弹出键盘以后，输入框距离键盘有一段距离，原本输入框之下有一个按钮，现在使用一个新的Window以后，可以在布局上去掉这按钮了，那么输入框距离bottom的距离就可以随意调节了。在Dialog的Window就不需要关心布局变样了，因此可以使用adjustResize了。在上面的测试知道adjustResize只有在非全屏下使用才正常，全屏下都会变成adjustPan的效果了。而我的需求是全屏，所以adjustResize的情况不能使用了，即使使用了adjustResize也是会退变为adjustPan效果了。

那么方案就定了，使用Dialog，也就没了SurfaceView的困扰，效果就如3.1的测试，因为全屏，只能选择adjustPan，并且也不能使用那两个令人呕吐的奇葩属性组合。至于，如何实现需求的效果，让输入框距键盘的距离可调节，就只能使用EditText的paddingBottom属性了。

把之前的输入框替换成为TextView，然后点击以后显示Dialog，Dialog的代码如下：

{% highlight java %}

private void showInputDialog(Context context) {
    AlertDialog.Builder mBuilder = new AlertDialog.Builder(context, R.style.DialogFullscreen);
    Dialog mDialog = mBuilder.create();

    mDialog.setCancelable(true);
    mDialog.setCanceledOnTouchOutside(true);
    mDialog.show();
    mDialog.setContentView(R.layout.dialog_input_layout);

    Window window = mDialog.getWindow();
    window.clearFlags(WindowManager.LayoutParams.FLAG_ALT_FOCUSABLE_IM);
    window.setSoftInputMode(WindowManager.LayoutParams.SOFT_INPUT_STATE_ALWAYS_VISIBLE 
        | WindowManager.LayoutParams.SOFT_INPUT_ADJUST_PAN);
}

{% endhighlight %}

其中，dialog的DialogFullscreen的style如下：

{% highlight xml %}

<style name="DialogFullscreen">
    <item name="android:windowContentOverlay">@null</item>
    <!-- 边框 -->
    <item name="android:windowFrame">@null</item>
    <!-- 半透明 -->
    <item name="android:windowIsTranslucent">true</item>
    <!-- 背景透明 -->
    <item name="android:windowBackground">@android:color/transparent</item>
    <item name="android:windowFullscreen">true</item>
    <item name="android:windowNoTitle">true</item>
    <item name="android:backgroundDimEnabled">true</item>
    <item name="android:backgroundDimAmount">0.5</item>
</style>

{% endhighlight %}

而布局就只有一个输入框了，最终效果如下图所示:

![alt keyboard](/image/edittext_keyboard_10.png "keyboard")

## 5. 结案陈词

到现在，输入框和键盘的血案也是可以结案陈词了。正常使用，Google是不建议全屏的，如果全屏了的话，那么就是adjustResize失效，再加上SurfaceView或者WebView，就坑爹了，还有奇葩的属性也呕吐了，这都是Google的bug。反正还会有更多你意想不到的情况出现。尽量避免使用那些坑爹的属性组合。

如果已经是adjustPan的效果了，再使用AndroidBug5497Workaround调整布局大小的话，布局显示就会异常。只要在输入框被键盘覆盖的情况下使用才有效。

使用Dialog，可以解决SurfaceView的问题，但是全屏以后，还是用不了adjustResize的，只能用paddingBottom解决。