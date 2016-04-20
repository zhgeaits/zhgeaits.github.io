---
layout: post
title:  "网站开发需要学习的第一门技术HTML"
date:   2011-03-09 11:00:03
categories: frontend
supertype: career
type: java&web
---

>html css和js的学习网站首选推荐梦之都：http://www.dreamdu.com/

其实当年学习完JavaSE以后就转到了网站开发，也就是JavaEE的路线来了（不然我就搞Android去了），然后搞网站一个学习的就是html了，哈哈，依旧是看各种视频。

## 1.什么是HTML和XHTML

HTML是Hypertext Markup Language的英文缩写，即超文本标记语言，由W3C的维护的。它是一种标记语言，不需要编译，直接由浏览器执行，也就是说，其实它只是一个标准的语法罢了，而浏览器就是实现解析了这个规范。

XHTML是EXtensible HyperText Markup Language的英文缩写，即可扩展的超文本标记语言。不过是一种增强了的HTML，它的可扩展性和灵活性将适应未来网络应用更多的需求。也可以说就是HTML一个升级版本(w3c描述它为'HTML 4.01'，呵呵，那其实html5也不过是新的规范罢了，支持更多牛逼的功能)。XHTML是大小写敏感的，HTML与html是不一样的，标准的XHTML标签应该使用小写。

HTML的结构包括：文档(页面)，元素，标签和属性，其实最重要就是学会标签和属性就可以了。

## 2.HTML的属性

属性分为两种：一般属性(common)和私有属性。

一般属性包括以下四种：

### 2.1 核心属性(core)

1.xml:space：这个属性还没用过，取值好像是"default"* | "preserve"。  
2.class：css的类名。  
3.id：就是id值。   
4.title：鼠标移到上面会显示的。

### 2.2 I18N属性(Internationalization)

1.xml:lang : XHTML文档，元素中内容的语言代码。  
2.dir：文字的方向，取值为"ltr" | "rtl"。  
3.lang：HTML文档，元素中内容的语言代码。

### 2.3 事件属性(events)

1.onclick  
2.ondblclick  
3.onmousedown  
4.onmouseup  
5.onmouseover  
6.onmousemove  
7.onmouseout  
8.onkeypress  
9.onkeydown  
10.onkeyup 

### 2.4 样式属性(style)

1.style：元素的内联样式，填的就是css代码

其实就是common = Core + Events + I18N + Style，大多数标签都具有"一般属性",有标签还具有自己的"私有属性"。比如链接的href属性，对于私有属性，在具体学习每个标签的时候会分别介绍的。

## 3.HTML标签

html有大量的标签，这里只记录了一些常用的，以后会在慢慢添加记录进来。

### 3.1文档开始标签`<html></html>`

属性：

<pre>
Common -- 一般属性
xml:lang -- 国际化属性
xmlns -- 代表xml命名空间
dir -- 定义元素(文字)的对齐方式
</pre>

### 3.2头部标签`<head></head>` 

头(head)包含了当前文档的一些信息，例如标题信息，meta信息等，正常情况下头信息是不会显示在HTML文档中的。
head具有一般属性；head元素包含的标签有：title，base，link，style，script，meta。

### 3.3标题`<title></title>`

title的内容通常在浏览器的标题栏中显示，浏览器中收藏夹内书签的名称是title的内容，title的内容可以方便搜索引擎索引页面，从搜索引擎搜索到的内容的标题往往是网页title的内容。

### 3.4主体标签`<body></body>`

比较特别的有onload属性。

### 3.5换行`<br />` 

line break的意思，具有一般属性和clear属性（换行输出方式），标准网页设计中完全可以不使用br，应尽量避免使用br标签。

### 3.6水平线`<hr />` 

这个单词的缩写：horizontal rule;有以下属性:

<pre>
common -- 一般属性
align -- 对齐方式，应使用CSS实现
noshade -- 定义显示方式(不定义为原始颜色，定义了为带凹槽的样式)，应使用CSS实现
size -- 定义水平线高度，应使用CSS实现
width -- 定义水平线宽度，应使用CSS实现
</pre>

### 3.7超链接`<a href="">`

a是anchor(锚)的缩写，有以下属性：

<pre>
common -- 一般属性
accesskey -- 代表一个链接的快捷键访问方式
charset -- 指定了链接到的页面所使用的编码方式,比如UTF-8
coords -- 使用图像地图的时候可以使用此属性定义链接的区域,通常是使用x,y坐标
href -- 代表一个链接源(就是链接到什么地方)
hreflang -- 指出了链接到的页面所使用的语言编码
rel -- 代表文档与链接到的内容(href所指的内容)的关系
rev -- 代表文档与链接到的内容(href所指的内容)的关系
shape -- 使用图像地图的时候可以使用shape指定链接区域
tabindex -- 代表使用"tab"键,遍历链接的顺序
target -- 用来指出哪个窗口或框架应该被此链接打开
title -- 代表链接的附加提示信息
type -- 代表链接的MIME类型
</pre>

以上属性还可以继续完善，href还可以加#id来链接到指定的元素，如`href="www.xhomestudio.org#top"`;还链接到邮件：`href="mailto:330810851@qq.com"`。

### 3.8载入图片`<img src="" />` 

属性:

<pre>
Common -- 一般属性
alt -- 代表图像的替代文字
src -- 代表一个图像源(就是图像的位置)
height -- 代表一个图像的高度
width -- 代表一个图像的宽度
</pre>

### 3.9原点列表`<ul></ul>`

这是unordered lists的缩写，样式是原点，li是list item的缩写，例子如下：
{% highlight html %}
<ul>
	<li>www</li>
	<li>dreamdu</li>
	<li>com</li>
</ul>
{% endhighlight %}

### 3.10数字顺序列表`<ol></ol>`

这是ordered lists的缩写,样式是数字顺序，例子如下：
{% highlight html %}
<ol>
	<li>www</li>
	<li>dreamdu</li>
	<li>com</li>
</ol>
{% endhighlight %}

### 3.11定义列表`<dl></dl>`

这是definition lists的英文缩写，中文"定义列表"的意思，自定义列表的开始使用<dl>标签,列表中每个元素的标题使用<dt>(definition term)定义，后面跟随<dd>(definition description)用于描述列表中元素的内容。例子如下：

{% highlight html %}
<dl>
    <dt>www</dt>
            <dd>World Wide Web的缩写.</dd>
    <dt>dreamdu</dt>
            <dd>梦之都.</dd>
            <dd>www的:).</dd>
    <dt>com</dt>
    <dt>com.cn</dt>
    <dt>cn</dt>
            <dd>这都是域名的后缀.</dd>
</dl>
{% endhighlight %}

### 3.12导航列表`<nl></nl>`

这是navigation lists的英文缩写，中文"导航列表"的意思，例子：
{% highlight html %}
<nl>
         <label>梦之都</label>
         <li href="#introduction">介绍</li>
         <li>
         <nl>
                 <label>网址(http://www.dreamdu.com/)</label>
                 <li href="#http">http://</li>
                 <li href="#www">www</li>
                 <li href="#dreamdu">dreamdu</li>
                 <li href="#com">com</li>
         </nl>
         </li>
         <li href="#html">html教程</li>
         <li href="#css">css教程</li>
</nl>
{% endhighlight %}

### 3.13表格`<table></table>`

标题标签<caption></caption>

**表格行标签：<tr></tr>;"table row"的缩写;属性如下：**

<pre>
align -- 代表行的水平对齐方式(left(左对齐) | center(居中对齐) |right(右对齐) | justify)(此属性应该使用CSS实现)
valign -- 代表行的垂直对齐方式(top(顶部对齐) | middle(中部对齐) | bottom(下部对齐) | baseline(基线对齐))(此属性应该使用CSS实现)
</pre>

**表头标签：<th></th>"table header cell"的缩写，属性如下：**

<pre>
abbr -- 代表表头的简写
axis -- 对单元格在概念上分类
colspan -- 一行跨越多列
headers -- 连接表格的数据与表头
rowspan -- 一列跨越多行
scope -- 定义行或列的表头
align -- 代表水平对齐方式(left(左对齐) | center(居中对齐) | right(右对齐) | justify)(此属性应该使用CSS实现)
valign -- 代表垂直对齐方式(top(顶部对齐) | middle(中部对齐) | bottom(下部对齐) | baseline(基线对齐))(此属性应该使用CSS实现)
</pre>

**表列标签:<td></td>"table data cell"的缩写,属性如下：**

<pre>
abbr -- 代表表头的简写
axis -- 对单元格在概念上分类
colspan -- 一行跨越多列
headers -- 连接表格的数据与表头
rowspan -- 一列跨越多行
scope -- 定义行或列的表头
align -- 代表水平对齐方式(left(左对齐) | center(居中对齐) | right(右对齐) | justify)(此属性应该使用CSS实现)
valign -- 代表垂直对齐方式(top(顶部对齐) | middle(中部对齐) | bottom(下部对齐) | baseline(基线对齐))(此属性应该使用CSS实现)
</pre>

**colgroup标签**：`<colgroup></colgroup>`表示对HTML表格进行结构化的分区,在此分区中可以通过使用col定义每列表格的样式。

### 3.14form标签：`<form></form>`

属性如下：
<pre>
action -- 浏览者输入的数据被传送到的地方,比如一个PHP页面(dreamdu.php)
method -- 数据传送的方法
enctype -- 表示将数据发送到服务器时浏览器使用的编码类型
</pre>

### 3.15`<input/>`标签

属性如下：
<pre>
type -- 代表一个输入域的显示方式(分为输入型,选择型,点击型)
value -- 输入域的值
size -- 输入域的长度
maxlength -- 输入域最多可以输入文字的长度
checked -- 如果是选择型的输入域,代表已经被选择
readonly -- 输入域可以选择,但是无法修改
disabled -- 输入域无法获得焦点,无法选择,以灰色显示,在表单中不起任何作用
accesskey -- 表单的快捷键访问方式
tabindex -- 输入域的"tab"键遍历顺序
src -- 当使用图片来表示按钮时,代表图片的位置(URI)
alt -- 用来替换提交按钮的图片(当在input的src属性定义的图片无法显示时)
</pre>

**其中type的取值如下：**
<pre>
text -- 文字输入域(输入型)
password -- 也是文字输入域,但是输入的文字以密码符号'*'显示(输入型)
file -- 可以输入一个文件路径(输入型)
checkbox -- 复选框.可以选择零个或多个(选择型)
radio -- 单选框.只可以选择一个而且必须选择一个(选择型)
hidden -- 代表隐藏域,可以传送一些隐藏的信息到服务器
button -- 按钮(点击型)
image -- 使用图片来显示按钮,使用src属性指定图像的位置(就像img标签的src屬性)(点击型)
submit -- 提交按钮,表单填写完毕可以提交,把信息传送到服务器.可以使用value属性來显示按鈕上的文字(点击型)
reset -- 重置按钮,可以把表单中的信息清空(点击型)
</pre>

### 3.16`<textarea>`标签

属性如下：
<pre>
cols -- 多行输入域的列数
rows -- 多行输入域的行数
accesskey -- 表单的快捷键访问方式
disabled -- 输入域无法获得焦点,无法选择,以灰色显示,在表单中不起任何作用
readonly -- 输入域可以选择,但是无法修改
tabindex -- 输入域,使用"tab"键的遍历顺序
</pre>

### 3.17`<select>`标签

属性如下：
<pre>
size -- 选择域的高度
multiple -- 可以有多个选择
disabled -- 输入框无法获得焦点,无法选择,以灰色显示,在表单中不起任何作用
tabindex -- 使用"tab"键的遍历顺序
</pre>

**select的子标签`<option>`标签，属性如下：**
<pre>
label -- 说明选择项
value -- 说明选择项的值
selected -- 此选择项已经被选择
disabled -- 输入框无法获得焦点,无法选择,以灰色显示,在表单中不起任何作用
tabindex -- 使用"tab"键的遍历顺序
</pre>

**select的子标签`<optgroup>`标签，它是不能选择的，属性如下：**
<pre>
label -- 说明选择项
</pre>

### 3.18`<label>`标签

提示表单的含义，属性如下：
<pre>
for -- 与此label描述的表单的id
accesskey -- 表单的快捷键访问方式
</pre>

### 3.19`<fieldset>`标签和`<legend>`标签

如果一个页面的表单项太多,我们最好把它们分组显示,就像使用p标签分开段落一样,可以使用fieldset与legend标签对表单内容分组.
一个表单可以有多个<fieldset>,每对<fieldset>为一组,每组内容的描述可以使用legend标签说明。

### 3.20简称标签`<abbr></abbr>`

abbreviation的缩写，例子：`<abbr title="Limited">Ltd.</abbr>`

### 3.21取首字母的缩写标签`<acronym></acronym>标签`

例子：`<acronym title="Cascading Style Sheets">CSS</acronym>`

### 3.22引用标签`<cite></cite>`

例子：`<cite cite="http://www.dreamdu.com/xhtml/">一步步的教我学会HTML与XHTML</cite>`

### 3.23代码标签`<code></code>`

例子：PHP中的变量名,前面要加上 '$' 符号 例如: `<code>$i = 1;</code>`

### 3.24定义标签`<dfn></dfn>`

例子：`<dfn>梦之都</dfn>`是一个单词,更是一种向往!

### 3.25斜体标签`<em></em>`

emphasis的缩写，明明是强调的意思，确实斜体的效果。不过还是建议使用css的font-style属性

### 3.26使用者输入的文字标签`<kbd></kbd>`

例子：To exit, type `<kbd>QUIT</kbd>`.

### 3.27语句标签`<l></l>`

line of text,例子：`<l>一行实实在在的文字!</l>`

### 3.28行引用标签`<q></q>`

quoted text的意思，例子：`<cite>古人</cite>`云:`<q>良言一句三冬暖,恶语伤人六月寒.</q>`

### 3.29程序或脚本输出的内容标签`<samp></samp>`

意思是sample output from programs,scripts,etc,例子：程序的输出是`<samp>x+y</samp>`

### 3.30内联行标签`<span></span>`

span表示了内联行,并无实际的意义,主要通过CSS样式(style)为其赋予不同的表现.

### 3.31重点加强标签`<strong></strong>`

不过建议使用css的font-weight属性

### 3.32下标标签`<sub></sub>`

subscript的意思

### 3.33上标标签`<sup></sup>`

supscript的意思

### 3.34程序中的变量标签`<var></var>`

variable的意思，例子：变量`<var>$i</var>`,代表循环次数.

### 3.35粗体标签`<b>`

### 3.36斜体标签`<i>`

### 3.37`<div>`标签

就像span标签一样,和CSS联合起来才能显示出它的威力，如果把html比作C/S中的窗体，DIV就是窗体中的面板。面板更好的管理控件，更好地布局，对权限的实现提供了一些支持。

### 3.38标题标签`<h></h>`

一般表示文章主标题，包含h1-h6六种大小，是从大到小的顺序的,h1最大,h6最小

### 3.39段落`<p></p>`

paragraph的缩写，具有一般属性

### 3.40地址标签`<address></address>`

例子：`<address href="mailto:dreamdu@dreamdu.com">梦之都 webmaster</address>`

### 3.41程序代码块标签`<blockcode></blockcode>`

此标签与pre标签很相似,可以表现出空格,回车,tab键。例子：

{% highlight html %}
<blockcode class="PHP" cite="dreamdu">
function dreamdu()
{
        $dreamdu = "www.dreamdu.com";
        echo $dreamdu;
}
</blockcode>
{% endhighlight %}

### 3.42引用块标签`<blockquote></blockquote>`

例子：
{% highlight html %}
<blockquote cite="http://www.dreamdu.com/xhtml/">
        <p>标准网页设计要区分内容与表现,学习标准网页设计.</p>
</blockquote>
{% endhighlight %}

### 3.43标签`<pre></pre>`

按原文显示标签，可以把原文件中的空格,回车,换行,tab键表现出来

### 3.44标签`<section></section>`

一般联合h标签一起使用,表示一部分

### 3.45文档类型标签`<!DOCTYPE >`

DOCTYPE，简称为DTDs 例如：

{% highlight html %}
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN" " http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd ">
{% endhighlight %}

http://www.dreamdu.com/xhtml/tag_doctype/这里有详细介绍各种文档类型，HTML声明中使用的标签是很特殊的(不同于前面介绍的标签语法)，使用<!开始，结束也不用关闭符。

### 3.46标签`<meta/>`

metainformation的缩写，属性如下：

<pre>
I18N -- i18n属性
xml:lang -- 国际化属性
content -- content属性
http-equiv -- http-equiv属性
id -- id属性
name -- name属性
scheme -- scheme属性
</pre>

**meta属性主要分为两组**

1.name属性与content属性

name属性用于描述网页，它是以名称/值形式的名称，name属性的值所描述的内容(值)通过content属性表示，便于搜索引擎机器人查找，分类。其中最重要的是description，keywords和robots。

2.http-equiv属性与content属性

http-equiv属性用于提供HTTP协议的响应头报文(MIME文档头)，它是以名称/值形式的名称，http-equiv属性的值所描述的内容(值)通过content属性表示，通常为网页加载前提供给浏览器等设备使用。其中最重要的是content-type charset 提供编码信息，refresh刷新与跳转页面，no-cache 页面缓存，expires网页缓存过期时间。

**content-type 属性值：**

content-type用于定义用户的浏览器或相关设备如何显示将要加载的数据，或者如何处理将要加载的数据，此属性的值可以查看MIME类型。使用content属性表示页面的MIME类型

例子：
{% highlight html %}
<meta http-equiv="content-type" content="text/html; charset=UTF-8" />
{% endhighlight %}

**content-language**

content-language用于定义页面所使用的语言代码，使用content属性表示页面的语言以及国家代码。

例子：
{% highlight html %}
<meta http-equiv="content-language" content="zh-CN" />
{% endhighlight %}

**refresh**

例子
{% highlight html %}
<meta http-equiv="refresh" content="5; url=http://www.dreamdu.com/" />
{% endhighlight %}

**Expiress属性值：**

expires用于设定网页的过期时间，一旦过期就必须从服务器上重新加载。时间必须使用GMT格式。例子：

{% highlight html %}
<meta http-equiv="expires" content="Sunday 26 October 2008 01:00 GMT" />
{% endhighlight %}

**pragma与no-cache 属性值 -- 定义页面缓存：**

不缓存页面(为了提高速度一些浏览器会缓存浏览者浏览过的页面，通过下面的定义，浏览器一般不会缓存页面，而且浏览器无法脱机浏览)。例子：

{% highlight html %}
<meta http-equiv="pragma" content="no-cache" />
{% endhighlight %}

**keywords 属性值 -- 定义网页关键词**

{% highlight html %}
<meta name="keywords" content="HTML XHTML" />
{% endhighlight %}

**description 属性值 -- 定义网页简短描述**

{% highlight html %}
<meta name="description" content="html toturial and html books" />
{% endhighlight %}

**author 属性值 -- 定义网页作者**

{% highlight html %}
<meta name="author" content="http://www.dreamdu.com/blog/" />
{% endhighlight %}

**copyright 属性值 -- 定义网页版权**

{% highlight html %}
<meta name="copyright" content="© http://www.dreamdu.com" />
{% endhighlight %}

**date 属性值 -- 定义网页生成时间**

{% highlight html %}
<meta name="date" content="2008-07-12T20:50:30+00:00" />
{% endhighlight %}

**robots 属性值 -- 定义网页搜索引擎索引方式**

robots用于定义网页搜索引擎索引方式，robots出现在name属性中，使用content属性定义网页搜索引擎索引方式。通过使用meta的robots属性值可以为没有文件上传权限的用户提供/robots.txt文件的所有功能。

{% highlight html %}
<meta name="robots" content="robotterms" />
{% endhighlight %}

robotterms是一组使用逗号(,)分割的值，通常有如下几种取值：none，noindex，nofollow，all，index和follow。
<pre>
none
搜索引擎将忽略此网页，等价于noindex，nofollow。
noindex
搜索引擎不索引此网页。
nofollow
搜索引擎不继续通过此网页的链接索引搜索其它的网页。
all
搜索引擎将索引此网页与继续通过此网页的链接索引，等价于index，follow。
index
搜索引擎索引此网页。
follow
搜索引擎继续通过此网页的链接索引搜索其它的网页。
</pre>

### 3.47`<style>`标签

属性如下：
<pre>
media -- 媒体类型,参见CSS高级教程
type -- 包含内容的类型,一般使用type="text/css"
</pre>

### 3.48`<link>`标签

属性如下：
<pre>
href -- 指定需要加载的资源(CSS文件)的地址URI
media -- 媒体类型
rel -- 指定链接类型
rev -- 指定链接类型
title -- 指定元素名称
type -- 包含内容的类型,一般使用type="text/css"
</pre>

例如：

{% highlight html %}
<link rel="stylesheet" type="text/css" href="style.css" />
{% endhighlight %}

**rel属性：**

rel属性通常出现在a，link标签中，可选值：

<pre>
alternate -- 定义交替出现的链接
appendix -- 定义文档的附加信息
bookmark -- 书签
canonical -- 规范网页是一组内容高度相似的网页的首选版本
chapter -- 当前文档的章节
contents
copyright -- 当前文档的版权
glossary -- 词汇
help -- 链接帮助信息
index -- 当前文档的索引
next -- 记录文档的下一页.(浏览器可以提前加载此页)
nofollow -- 不被用于计算PageRank
prev -- 记录文档的上一页.(定义浏览器的后退键)
section -- 作为文档的一部分
start -- 通知搜索引擎,文档的开始
stylesheet -- 定义一个外部加载的样式表
subsection -- 作为文档的一小部分
rel与rev具有互补的作用,rel指定了向前链接的关系,rev指定了反向链接的关系.
</pre>

**rev 属性**

reverse link，描述了href所指定文档与当前页面的关系。rev属性通常出现在a，link标签中，可选值：

<pre>
alternate -- 定义交替出现的链接
appendix -- 定义文档的附加信息
bookmark -- 书签
chapter -- 当前文档的章节
contents
copyright -- 当前文档的版权
glossary -- 词汇
help -- 链接帮助信息
index -- 当前文档的索引
next -- 记录文档的下一页.(浏览器可以提前加载此页)
nofollow -- 不被用于计算PageRank
prev -- 记录文档的上一页.(定义浏览器的后退键)
section -- 作为文档的一部分
start -- 通知搜索引擎,文档的开始
stylesheet -- 定义一个外部加载的样式表
subsection -- 作为文档的一小部分
</pre>

### 3.49脚本标签`<script></script>`

属性：
<pre>
src -- 指定需要加载的脚本文件(例如:js文件)的地址URI
type -- 指定媒体类型(例如:type="text/javascript")
</pre>

### 3.50 rea标签

area标签必须使用在map标签中，而且必须配合img标签使用。属性：

<pre>
accesskey -- 链接的快捷键访问方式
alt -- 图像的提示文字
coords -- 定义可点击区域图形的坐标
href -- HTML链接源的URL
nohref -- 图像点击排除的区域，当不使用href时应使用nohref
shape -- 可点击区域的形状
tabindex -- 使用"Tab"键的遍历顺序
target -- 链接目标
</pre>

例子：
{% highlight html %}
<img src="http://www.dreamdu.com/images/html_table.png" usemap="#Map" />
<map name="Map" id="Map">
        <area shape="rect" coords="35,29,135,99" href="#" />
        <area shape="circle" coords="243,78,44" href="#" />
        <area shape="poly" coords="120,137,195,154,135,207" href="#" />
</map>
{% endhighlight %}

### 3.51 map标签

map标签中name与id属性指定的值必须与定义图像点击区中图像(img)的usemap属性指定的值一致。

### 3.52 base标签

base标签只能放置在head标签内，定义基URL用于页面的链接与引用，当使用相对路径定义链接时，可以使用base标签定义基URL解析所有文档中定义的相对路径的URL。

### 3.53 button标签

属性：
<pre>
accesskey -- 快捷键访问方式
disabled -- 禁止使用
name -- 标签名称
tabindex -- 使用"Tab"键的遍历顺序
type -- 按钮类型button -- 普通按钮 reset -- 重置表单按钮submit -- 提交按钮
value -- 通过表单传递到服务器端的数据
</pre>

### 3.53 param标签

定义为object标签提供嵌入内容的运行时参数的name与value对。属性：

<pre>
id -- 唯一标识符
name -- 名称，name与value属性组成一对，见下面示例
type -- 嵌入内容的MIME类型
value -- 相对与name的值
valuetype -- 指定参数类型
data -- 参数是一个简单的字符串，默认值
ref -- 参数是URL
data -- 参数是另一个嵌入式对象
</pre>

### 3.54 object标签

使用object标签可以在网页中嵌入各种多媒体，例如Flash,Java Applets,MP3,QuickTime Movies等。param标签通常配合object标签一同使用；object标签内除了param标签外，其它的内容将在浏览器不支持objcet标签时显示。

属性：
<pre>
archive -- 包含多个使用逗号(,)分割的Java类或外部资源，用于增强applet的功能，定义applet代码
border -- 边框
classid -- 关联一个应用程序，执行嵌入内容的应用程序在windows系统中的唯一id(不能改变此id，否则程序将出现异常)
codebase -- 为相对路径提供基URL，IE浏览器通常将此属性中的内容定义为运行嵌入内容所要加载的插件
codetype -- 嵌入内容的MIME类型
declare -- 声明没有实例化的嵌入内容，此内容通常在加载后可以使用，或者当嵌入内容的某些参数将使用其它嵌入内容时
declare -- 声明
data -- 嵌入内容的URL，可以是根据codebase属性的相对路径
height -- 嵌入内容的高度，单位像素
hspace -- 嵌入内容水平方向的空白，应使用CSS margin代替
standby -- 文档加载时显示的文本信息
tabindex -- 使用"Tab"键的遍历顺序
usemap -- 定义图像点击区域
viewastext -- 阻止一些可视化工具，例如Microsoft FrontPage或者Visual InterDev在设计页面时运行object内的组件
vspace -- 嵌入内容垂直方向的空白，应使用CSS margin代替
width -- 嵌入内容的宽度，单位像素
</pre>

例子：
{% highlight html %}
<object classid="clsid:D27CDB6E-AE6D-11cf-96B8-444553540000" codebase="http://download.macromedia.com/pub/shockwave/cabs/flash/swflash.cab#version=7,0,19,0">
        您的浏览器不支持Flash，请点击<a href="http://www.macromedia.com/go/getflashplayer">下载</a>。
        <param name="movie" value="/1.swf" />
        <param name="quality" value="high" />
</object>
{% endhighlight %}

## 4.特殊字符

`&quot;` -- 表示双引号"

`&amp;` -- 表示位与运算符&

`&lt;` -- 表示小于运算符<

`&gt;` -- 表示大于运算符>

`&nbsp;` -- 表示空格

`&copy;` -- 代表©

`&reg;` -- 代表®
