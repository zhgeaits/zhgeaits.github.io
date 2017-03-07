---
layout: post
title:  "Learning an Android Debug Tool--Stetho"
date:   2017-03-07 11:00:03
categories: android
supertype: career
type: android
---

## 1. What's Stetho

官方[地址在这](http://facebook.github.io/stetho/)，还有[GitHub](https://github.com/facebook/stetho)。

这简直就是一个开发神器！

官网上面介绍，它是一个精美的连接Chrome的开发调试工具。开发者可以通过Chrome浏览器进行连接手机进行调试，具体的功能包括：网络抓包，数据库视图操作，View层次结构，dumpapp和执行js。特别是数据库这个，就像以前装了一个mysql的navicat一样，终于不用在命令行里面一行行的操作了，麻烦死了。

安装使用的方法，官网都具备了，这里就不搬砖了。在chrome里面进入chrome://inspect即可。需要注意的是，如果不关闭这个页面，studio就会跟手机断开连接的了。

## 2.开发

如果本地以及依赖了OkHttp那么就会出现包冲突了，所以要这样配置：

{% highlight shell %}

debugCompile 'com.facebook.stetho:stetho:1.4.2'
debugCompile ('com.facebook.stetho:stetho-okhttp3:1.4.2') {
    exclude module: 'okio'
    exclude module: 'okhttp'
}

{% endhighlight %}

如果方法数已经超了，则需要multiDex：

{% highlight shell %}

defaultConfig {
    multiDexEnabled = true
}

{% endhighlight %}

由于我们app对api的最低要求是api 8，但是stetho的最低要求是9，所以需要在AndroidManifest里面配置一下，强制要求stetho也是9：

>tools:overrideLibrary="com.facebook.stetho,com.facebook.stetho.okhttp3"

由于我们是maven构建打包的，本地开发使用的确实gradle，所以，只有在开发环境才配置stetho，于是用反射来安装stetho：

{% highlight java %}

public class StethoUtils {

    public static void installStetho(Context context) {
        if (RunTime.gIsDebug) {
            try {
                Class<? extends Object> clazz = Class.forName("com.facebook.stetho.Stetho");
                Method installMethod = clazz.getMethod("initializeWithDefaults", Context.class);
                installMethod.invoke(null, context);

                Log.i("StethoUtils", "installStetho install stetho finished.");
            } catch (Throwable e) {
                Log.e("StethoUtils", "installStetho error:" + e.getMessage());
            }
        }
    }

    public static void addOkHttpInterceptor(OkHttpClient.Builder builder) {
        if (RunTime.gIsDebug) {
            try {
                Class<? extends Object> clazz = Class.forName("com.facebook.stetho.okhttp3.StethoInterceptor");
                Interceptor interceptor = (Interceptor) clazz.newInstance();
                builder.addInterceptor(interceptor);

                Log.i("StethoUtils", "addOkHttpInterceptor finished.");
            } catch (Throwable e) {
                Log.e("StethoUtils", "addOkHttpInterceptor error:" + e.getMessage());
            }
        }
    }
}

{% endhighlight %}
   
## 2.数据库视图

以前调试数据库，如果手机没有root的话，是很麻烦的，详细看这篇[blog]()。

现在可以直接在chrome里面可视化看到数据库表。并且可以执行sql语句。但是ui还是不是很强大，如果以后能做到想navicat那样就好了。

## 3.UI树状结构

直接可以在第一个tab的Elements里面查看到整个app的view结构。最终要的一点是，它的根节点是Application，一直往下，包括了window，这非常利于我们的学习，重复告诉我们app的UI view结构。详细关于View的看这篇[blog](http://zhgeaits.me/android/2014/07/27/android-ui-view-study-notes.html)。关键是，还是可以简单修改view的值啊，直接就是可视化的开发界面了！

## 4.网络抓包

我使用的网络库是OkHttp，详细看这篇[blog](http://zhgeaits.me/android/2017/02/23/android-okhttp.html)。以前我们是用fiddler或者charles，只需要配置代理即可。虽然把所有的包都抓了，但是很多，我们这里是只抓自己app的包，只要app里面都走OkHttp就可以了。实际上使用就是调用上面的方法添加一个拦截器即可。现在是极其方便，甚至连对于网络响应的数据都不用打Log了。

## 5.执行JS

这点我还没去使用。