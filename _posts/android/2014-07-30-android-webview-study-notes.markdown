---
layout: post
title:  "Android的WebView, JavaScript学习笔记!"
date:   2014-07-30 11:00:03
categories: android
type: android
---

webview的使用很简单，设置网络权限，控件，然后loadurl就OK：  
{% highlight java %}
webView.getSettings().setJavaScriptEnabled(true);
webView.getSettings().setBuiltInZoomControls(true);
webView.loadUrl("http://www.baidu.com");  

//然后重写一个方法来捕获后退事件：  
//默认点回退键，会退出Activity，需监听按键操作，使回退在WebView内发生
@Override
public boolean onKeyDown(int keyCode, KeyEvent event) {  
	// TODO Auto-generated method stub  
	if ((keyCode == KeyEvent.KEYCODE_BACK) && webView.canGoBack()) {  
		webView.goBack();  
		return true;  
	}  
	return super.onKeyDown(keyCode, event);  
}  
{% endhighlight %}

难点在于javascript的调用：  
loadUrl()方法可以加载url，也可以去调用页面的js：webView.loadUrl("javascript:test()"); 这个test()方法就是html页面的js函数。

然后是JS调用android的方法：  
webview.addJavascriptInterface(OBJ, interfacename)使用这个方法就可以了。  
OBJ就是让js调用android的对象，interfacename就是js调用时的对象名。例如：  
{% highlight java %}
webview.addJavascriptInterface(new JSCallback(), "testjs")

public class JSCallback {
	public void test(String parm) {
		//to-do
	}
}

//然后js那里这样调用
<script type="text/javascript">
testjs.test("haha");
</script>
{% endhighlight %}

由于没有做过想过的东西和业务，更加复杂的就没用过，只知道这些简单用法了。