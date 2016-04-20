---
layout: post
title:  "网站开发需要学习的第三门技术JavaScript"
date:   2011-03-11 11:00:03
categories: javaweb
supertype: career
type: java&web
---

## 1 什么是JavaScript

>html css和js的学习网站首选推荐梦之都：http://www.dreamdu.com/

说到JavaScript，肯定会想到Java，那么请问JavaScript和Java的关系？那就是雷峰塔和雷锋的关系一样。这只是一个笑话而已；那么什么是javascript，历史问题自行google，只说一句，它是由Netscape公司的LiveScript改名而来的。前面所学习的html和css分别是网页的骨架和皮肤，那么其实js就是网页的动作了，也就是说js的主要作用是让网页可以动态起来，这不仅仅是画面的变换切换和动画的效果，更重要的是它可以和服务器交互，并且和用户交互起来。因此，JavaScript是一种基于对象和事件驱动的客户端脚本语言，既然是脚本语言，那就是弱语言了，也是让浏览器进行解析执行的了。

## 2 基本语法

这个基本语法没什么好记录的了，只要学过高级语言，这些都是自动就会的了，或许我会记录一些比较特别的东西而已，不懂就直接google罢了，重要的是学习js的整体系统机制。

### 2.1 引用方式

引用的方式和css相似，也是有三种。

**1.内联引用**

例如：

	<input type="button" value="点我学习JS教程" onclick="alert('你点击了一个按钮');">
	<a href="#" onclick="return DreamduPopup('http://zhgeaits.me/')">请点击我，打开CSS教程</a>

**2.内部引用**

一般代码比较少的情况下方便使用，用script标签包围（script标签看html学习记录），设置script标签的属性languange="javascript"也是可以的：

	<script type="text/javascript">
    	document.write("zhangge!");
		//如果用<![CDATA[]]>为这代码，那么就可以使用><等这些符号了。
    </script>
	
**3.外部引用**

建议使用这种方式，那是比较好的代码风格；也是使用script标签来引入一个文件即可，为防止网页加载缓慢，也可以把非关键的JavaScript放到网页底部：

	<head>
    	<script type="text/javascript" src="dreamdu.js"></script>
	</head>

### 2.2 常用的重要对象

重要的对象包括：Boolean, date, Math, Number, String。

### 2.3 各浏览器的用法

chrome的控制台里面JSON用法：

JSON.parse 解析字符串成json对象
JSON.stringify 把json对象转换成字符串


## 3 JS的面向对象语法

因为它是脚本语言，所以它的面向对象，很奇葩，其实如果熟悉json的话，也不会觉得奇葩的，因为json其实也是一个对象，它们是相通的。

### 3.1 使用构造函数来定义对象

不像java那样先得定义一个class类，在里面定义属性和方法；js只需要直接写一个构造函数就可以了，函数名就是它的类名了，然后属性和方法都是添加一个就是定义一个了；例如下面的定义：

{% highlight javascript %}

function Site(name, url)
{
	this.url = url;
	this.name = name;
}

Site.prototype.show = function()
{
	return this.name+"的网址为："+this.url;
};

{% endhighlight %}

然后就可以直接这样使用了：

{% highlight javascript %}

var mysite = new Site();
alert(mysite.url);
alert(mysite.show());

{% endhighlight %}

实际上，也是可以这样来定义的，这就比较像json的风格了：

{% highlight javascript %}

var Site={};
Site.url="";
Site.show=function(){...;};

//使用的时候就不需要new了
alert(mysite.url);
alert(mysite.show());

{% endhighlight %}

### 3.2 使用json的方法来定义对象

以前做web项目的时候用jquery都是这样用的，所以很熟悉了。

{% highlight javascript %}

var site =
{
	URL : "zhgeaits.me",
	name : "张戈",
	say : function(){document.write(this.englishname+" say : hello world!")}
	model:
	{
		name:"haha",
		action:function(){alert("hello");}
	}
};

{% endhighlight %}

## 4 JS的BOM编程

比较重要就是这个了，理解了才能够写出js特性的代码，不然的话依然是写出一个基本功能的代码。

### 4.1 什么是BOM和DOM编程

BOM的意思是browser object model，简称浏览器对象模型，由一系列相关的对象构成，并且每个对象都提供了很多方法与属性，即提供了独立于内容而与浏览器窗口进行交互的对象，由于BOM主要用于管理窗口与窗口之间的通讯，因此其核心对象是window。

BOM包含一个名叫DOM的节点，每种对象模型都由一种层次结构组成，这种层次结构就像金字塔，DOM的顶层是document对象。DOM的意思是Document Object Model的简写，既文档对象模型，它由一系列对象组成，是访问、检索、修改XHTML文档内容与结构的标准方法。

如下图关系：

![alt text](/image/bomdom.png "bom_dom")

也就是说，了解这些对象是干什么能干什么的就可以了，因为有太多的方法了，只要用到以后再来记录即可。

### 4.2 window对象

它是最顶层的对象，可以随便用，随便访问全局变量，"点"它就可以了。它的主要功能是用来控制窗口的，而其实我们常用的`alert()`,`parseInt()`,`isNaN()`等方法都是属于它的，所以默认就是调用了window对象了的，它是隐式调用的。它还有很多方法和属性，例如status，open()，close()，setTimeout()，moveBy()，moveTo()，resizeBy()，resizeTo()，scrollTo()，scrollBy()等等。

### 4.3 frames对象

它是用来操控frameset这个html标签的，因为这个标签已经不用了，所以这个对象也相当于废弃了。

### 4.4 history对象

就是操控页面的历史的对象，length属性中记录缓存了多少个URL，go()函数是前进或后退指定的页面数，back()函数是后退一页，forward()函数是前进一页。

### 4.5 location对象

location用于获取或设置窗体的URL，并且可以用于解析URL，是BOM中最重要的对象之一。location既是window对象的属性又是document对象的属性，即`window.location==document.location`是true。

	hash 属性	--	返回URL中#符号后面的内容
	host 属性	--	返回域名
	hostname 属性	--	返回主域名
	href 属性	--	返回当前文档的完整URL或设置当前文档的URL
	pathname 属性	--	返回URL中域名后的部分
	port 属性	--	返回URL中的端口
	protocol 属性	--	返回URL中的协议
	search 属性	--	返回URL中的查询字符串
	assign()函数 --	设置当前文档的URL
	replace()函数 --	设置当前文档的URL，并在history对象的地址列表中删除这个URL
	reload()函数 --	重新载入当前文档(从server服务器端)
	toString()函数 --	返回location对象href属性当前的值

### 4.6 navigator对象

navigator对象通常用于检测浏览器与操作系统的版本。最重要的是userAgent属性，返回包含浏览器版本等信息的字符串，其次cookieEnabled也很重要，使用它可以判断用户浏览器是否开启cookie。

### 4.7 screen对象

screen对象用于获取用户的屏幕信息。

	availHeight 属性	--	窗口可以使用的屏幕高度，单位像素
	availWidth 属性	--	窗口可以使用的屏幕宽度，单位像素
	colorDepth 属性	--	用户浏览器表示的颜色位数，通常为32位(每像素的位数)
	pixelDepth 属性	--	用户浏览器表示的颜色位数，通常为32位(每像素的位数)（IE不支持此属性）
	height 属性	--	屏幕的高度，单位像素
	width 属性	--	屏幕的宽度，单位像素

### 4.8 document对象

document用于表现HTML页面当前窗体的内容，即是dom的顶层对象，也是我们最常用的对象了。它的方法有write()，writeIn()，open()，close()，属性有：

	cookie --	用户cookie
	title	--	当前页面title标签中定义的文字
	URL	--	当前页面的URL
	anchors	--	文档中所有锚(a name="aname")的集合
	forms	--	文档中所有form标签表示的内容的集合
	images	--	文档中所有image标签表示的内容的集合
	links	-- 文档中所有a(链接)标签表示的内容的集合

#### 4.8.1 DOM的节点

可以把DOM中的每个节点理解为是一个类，它们有一个共同的父类Node，类似Java中所有类默认继承Object，DOM中的每个节点都继承之Node。如img标签button标签for标签(类)等都共同继承自Node类。根据子类拥有父类的属性和方法，只要弄懂Node类的属性和方法，就可以操作所有html节点的属性和方法。更多信息可以GoogleNode节点。

node节点的常用属性：documentElement，attributes，childNodes，firstChild，lastChild，nextSibling，previousSibling，parentNode，nodeType，nodeName，nodeValue。

node节点的常用方法：appendChild，setAttribute，removeChild，removeAttribute，replaceChild，createElement，createTextNode，insertBefore，cloneNode，hasChildNodes。

## 5 JQuery

Ajax提交格式，详细参数可以google：

	$.ajax({
		url:"../diagnosis/commit.do",	
		type:"POST",
		dataType:'json',
		contentType:"application/json",
		data:JSON.stringify(exam_answers),
		success:function(response){
			if(response.status==0){
				alert("提交成功")
			}else{
				alert("提交失败!");
			}	
		},
		error:function(){
			alert("ajax error");
		}
	});

Ajax异步提交form用到获取数据方法`$("#formid").serialize()`，serialize()方法通过序列化表单值，创建URL编码文本字符串。可以选择一个或多个表单元素（比如input或文本框），或者form元素本身。序列化的值可在生成AJAX请求时用于UR 查询字符串中。

这个提交的是text数据不是json数据，也可以自己修改转换成json再ajax。

	var frm = $(document.myform);
	var data = JSON.stringify(frm.serializeArray());
