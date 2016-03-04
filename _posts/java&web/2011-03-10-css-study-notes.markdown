---
layout: post
title:  "网站开发需要学习的第二门技术CSS"
date:   2011-03-10 11:00:03
categories: frontend
type: java&web
---

>html css和js的学习网站首选推荐梦之都：http://www.dreamdu.com/

其实学习后台是很痛苦的，不仅需要后端的技术，还需要掌握前端的技术，就算不去做前端开发，也是需要了解前端，所以，我们是很苦逼的。！~

## 1.什么是CSS

CSS是Cascading Style Sheets的英文缩写，即层叠样式表，由W3C的CSS工作组产生和维护的。它是一种标记语言，不需要编译，大小写不敏感的，可以直接由浏览器执行，用于布局与美化网页的。CSS文件是一个文本文件，它包含了一些CSS标记，CSS文件必须使用css为文件名后缀。

可以说html是网页的一个骨架，而css就是这个骨架的皮肤。

### 1.1引用css文件方式

那么如何在html里面编写css代码呢？有三种方式：

**1.内联引用**

学习html的时候我们知道标签都有styple的一般属性，那么如果想要修改某个标签的样式，可以直接在标签使用。例如，想要给p这个标签修改字体大小和颜色这个样式：

	<p style="font-size: 10px; color: #FFFFFF;">
        使用CSS内联引用表现段落.
	</p>

**2.内部引用**

学习html的时候知道有style这个标签，把这个标签的type属性设置为text/css就可以在这个标签里面编写css代码了。<![CDATA[]]>指的是不由 XML 解析器进行解析的文本数据。

	<style type="text/css">
		<![CDATA[
			在这里写
		]]>
	</style>


**3.外部引用**

这是最常用也是比较科学的方法，独立编写在一个css文件，然后引用进来。我们知道html有link这个标签，可以用来引用css文件，而link这个标签是在head标签之下的。

	<link rel="stylesheet" type="text/css" href="dreamdu.css" />

还可以使用@import引用方式在style标签引入：

	<style type="text/css">@import url(http://www.dreamdu.com/style.css);</style>

一般的浏览器都带有缓存功能，所以用户不用每次都下载CSS文件，外部引用相对于内部引用和内联引用来说是高效的是节省宽带的。

## 2.CSS基本语法

### 2.1基本语法

如何编写CSS代码，看下面一个模板：

	/* 这是注释 */
	选择符名字1，选择符名字2
	{
		声明;
		属性:属性值;
	}

注释不用说了，可以出现在任何地方；选择符名字就是我们想要对其编写代码的地方的代号，然后用两个大括号括起来，声明就是真正起作用的具体代码了，由"属性"，"冒号(:)"，"属性值"和"分号(;)"组成的。

### 2.2基本的选择符

#### 2.2.1选择符取名规则

CSS选择符可以使用英文字母的大写与小写(A-Z a-z)，数字(0-9)，连字号(-)，下划线(_)，冒号(:)和句号(.)组成。

#### 2.2.2常用的三种选择符

1.xhtml标签选择符，比如p标签选择符(代表所有的段落都使用这个CSS样式)，例如：

	p
	{
		font-size:12px;
	}

2.id选择符，唯一性选择符，就是在名字前增加了一个#，id选择符在一个页面中只能出现一次，在整个网站中也最好出现一次(这样有利于程序员控制此元素，有多个一样名称的元素，就无法分开不好控制了)。例如：

	#dreamdured
	{
		color:red;
	}

3.class选择符，多重选择符，就是在名字前增加了一个.，class选择符在一个页面中可以出现多次。一般一个标签有class属性，用到就是这个值。例如：

	.dreamdublue
	{
		color:blue;
	}

如果选择符的名字是*，则代表是全局选择符，所有东西都使用该样式。

### 2.3 数据类型

CSS的数值类型包含两种：整数（表示为<integer>）与实数（表示为<number>）。数值通常用于标明长度。整数前可以包括正(+)负(-)号，但是不能有小数。CSS的margin与padding都可以取负值。实数包含整数，而且可以有小数部分。例如1.5em，表示使用1.5倍字体的大小。

### 2.4 长度与单位

长度（表示为<length>），其公式为值是一个实数<number>，实数后面再加一个单位(例如:px,em等)，如果长度为0，也可以不加单位。很多属性可以使用负数的长度值，如果负数的长度值超出了CSS能容纳的限制，此长度值将被转变为可以支持的最接近的长度；如果某个属性不支持负数的长度，那么这个属性的声明将被忽略。

CSS长度与单位包含两种类型：相对与绝对。

绝对长度不依赖于环境（显示器、分辨率、操作系统等），相对长度依赖于用户使用的环境。绝对长度单位通常用于打印。

CSS支持的绝对长度单位：

	单位	英文	描述
	in	inches	英寸，英制单位，1英寸等于2.54厘米
	cm	centimeters	厘米，公制单位
	mm	millimeters	毫米，公制单位
	pt	points	点，1点等于1/72inches英寸，印刷设计中使用的单位
	pc	picas	铅字，1个pc等于12个point点，印刷设计中使用的单位


CSS支持的相对长度单位：

	单位	英文	描述
	em	无	依赖于最近的字体尺寸
	ex	无	依赖于英文子母小x的高度
	px	pixels	像素，依赖于用户的显示设备

em是依赖于最近的字体的大小，ex是相对于小写子母x的高度的倍数。ex测量单位被使用在排版中。px是pixel像素的缩写，这种测量方法依赖于用户计算机显示器的分辨率。

>网页设计中的字体大小经常使用px做为其单位，因为它很容易理解，而且图像的大小也使用px表示。由于px相对于显示器的实际大小，所以它可在显示器上提供适合的显示。px并不适合于文档的精确打印。

## 3.CSS高级语法

一般来说，上面的基本选择符就足够了，但是它还是有很多高级的用法的。高级选择符是由一个或多个简单选择符序列组成的链，多个简单选择符序列通过组合符分隔，且最后一个简单选择符序列可跟随一个伪元素。

组合符包括：空白（whitespace），>（大于号，greater-than sign），+（加号，plus sign），~（破折号，tilde）。

### 3.1类选择符高级用法

**1.链式使用**

类选择符名称可以使用多个.相连，形成类选择符链，例如：

	.www.dreamdu.com
	{
		color:blue;
	}

只有当一个元素使用了www、dreamdu、com三种选择符才能显示.www.dreamdu.com类选择符定义的样式。

**2.类选择符与属性值选择符**

HTML中的div.value与div[class~=value]是相同的。

	div.dreamdu-red
	{
		color: red;
	}
	div[class~=dreamdu-red]
	{
		color: red;
	}

上面的两个例子具有相同的含义，但是使用类选择符div.value更简洁明确。

### 3.2 ID选择符高级用法

HTML中的div.value与div[id=value]相同。

	div#dreamdu-red
	{
		color: red;
	}
	div[id=dreamdu-red]
	{
		color: red;
	}

### 3.3 组合选择符用法

**1.CSS包含选择符**

这个选择符是匹配文档中符合选择符规定的包含关系的元素，语法是：E F。

例如：

	div p
	{
		color: red;
	}

当p被div包含时（p是div的后裔时），p中文字的颜色为红色。当然也可以使用*，有多个后裔的时候，要注意组合，实际使用时候再测试便知道。

**2.CSS子对象选择符**

这个选择符是匹配文档中符合选择符规定的直接包含关系的元素，语法是：E > F。必须是直接包含关系，不能间接包含。

例如：

	p > span > code
	{
		color:green;
		font-size:80%;
	}

**3.CSS直接相邻选择符**

这个选择符是匹配文档中符合选择符规定的直接相邻关系的元素，语法是：E + F。

例如：

	h1 + h2
	{
		margin-top: -1em;
		color: green;
	}

**4.CSS普通相邻选择符**

这个选择符是匹配文档中符合选择符规定的普通相邻关系的元素，语法是：E ~ F。

例如，只有被同一个元素包含的p且p在span后面出现，则匹配选择符p：

	span ~ p
	{
		color:red;
	}

### 3.4 属性选择符用法

**1.CSS属性名选择符**

这个选择符是匹配文档中具有att属性的E元素，语法是：E[att]。

例如，所有具有title属性的a标签将显示红色的文字，并显示蓝色边框：

	a[title]
	{
		color:red;
		border: 1px solid blue;
	}

**2.CSS属性值选择符1**

这个选择符匹配文档中具有att属性且其值为val的E元素，语法是：E[att=val]。

例如：

	input[type="text"]
	{	
		background: green;
		color: white;
		border: 1px solid blue;
	}

**3.CSS属性值选择符2**

这个选择符匹配文档中具有att属性且其中一个值（多个值使用空格分隔）为val(val不能包含空格)的E元素，语法是：E[att~=val]。

例如（title是可以有多个值的）：

	a[title~="dreamdu"]
	{
		color:red;
	}

**4.CSS属性值选择符3**

这个选择符匹配文档中具有att属性且其中一个值为val，或者以val开头紧随其后的是连字符-的E元素（主要用来允许语言编码的匹配，例如HTML中的lang属性。语法是：E[att\|=val]。

例如（en, en-US）：

	*[lang|="en"]
	{
		color: red;
	}

**5.CSS属性值子串选择符1**

这个选择符匹配文档中具有att属性且其值的前缀为val的E元素。语法是：E[att^=val]。

例如：

	a[href^="https://"]
	{
		color:red;
		background: url(/images/css/icon-ssl.png) center right no-repeat;
	}

**6.CSS属性值子串选择符2**

这个选择符匹配文档中具有att属性且其值的后缀为val的E元素。语法是：E[att$=val]。

例如：

	a[href$=".html"]
	{
		color:red;
	}

**7.CSS属性值子串选择符3**

这个选择符匹配文档中具有att属性且其包含val的E元素。语法是：E[att*=val]。

例如：

	a[href*=".jsp"]
	{
		color:blue;
	}

### 3.5 伪元素

CSS中，样式和HTML文档中元素的连接通常基于元素在文档中的位置，这种方式满足于大部分需求。不过由于HTML文档结构的限制，一些效果无法实现，特定条件中，对于段落的第一行，没有一种简单的方法对其设置样式，但是可以使用伪元素::first-line设置段落第一行的样式。

CSS伪元素是CSS选择符的一部分，伪元素名称的大小写敏感性依赖于文档的语言，在HTML文档中大小写不敏感，在xml文档中大小写敏感。

语法：

::PseudoElementName

#### 3.5.1 CSS ::first-line 伪元素

这个伪元素匹配一个段落的第一行的样式，语法：E::first-line。例如：

	p::first-line
	{
		color: red;
	}

#### 3.5.2 CSS ::first-letter 伪元素

这个伪元素匹配第一个字母（中文是第一个文字）的样式，语法：E::first-letter。例如：

	p::first-letter
	{
		font-size: 4em;
		font-weight: bold;
		border: 1px solid lightblue;
		margin-right: 8px;
	}

#### 3.5.3 CSS ::before 伪元素

这个伪元素定义在一个元素的内容之前插入content属性定义的内容与样式，语法：E::before。例如：

	div::before
	{
		background: lightgreen;
		content: "张戈";
	}

#### 3.5.4 CSS ::after 伪元素，

这个伪元素定义在一个元素的内容之后插入content属性定义的内容与样式，语法：E::after。例如：

	div::after
	{
		background: lightgreen;
		content: "张戈";
	}

#### 3.5.5 CSS ::selection 伪元素

这个伪元素定义用户鼠标已选择内容的样式，语法：E::selection。例如：

	::selection
	{
		color:lightblue;
		background:pink;
	}

### 3.6 伪类

CSS中样式和HTML文档中元素的连接通常基于元素在文档中的位置，这种方式满足于大部分需求。不过由于HTML文档结构的限制，一些效果无法实现，例如，某些用户行为引发的事件：当用户鼠标移动到某个HTML元素上、离开HTML元素、点击HTML元素。伪类可以用于文档状态的改变、动态的事件等，例如用户的鼠标点击某个元素、未被访问的链接。伪类通过元素的名称、属性或内容三个特性对元素进行分类。原则上说是在HTML文档中无法获得的特性。CSS伪类是CSS选择符的一部分，伪类名称的大小写敏感敏感性依赖于文档的语言，在HTML文档中大小写不敏感，在xml文档中大小写敏感。

语法：

	:PseudoClasseName
	标签:PseudoClasseName
	标签.class:PseudoClasseName

#### 3.6.1 CSS :link 伪类

这个伪类适用于未被用户访问过的链接，例如：

	a:link
	{
		color: red;
	}

#### 3.6.2 CSS :visited 伪类，

这个伪类适用于已被用户访问过的链接，例如：

	a:visited
	{
		color: green;
	}

#### 3.6.3 CSS :hover 伪类

这个伪类适用于光标（鼠标指针）指向一个元素，但此元素未被激活时的样式，例如：

	a:hover 
	{
		color:yellow;
		background:blue;
		font-size:small;
	}

#### 3.6.4 CSS :active 伪类

这个伪类适用于一个元素被激活时（点击）的样式，例如：

	p:active
	{
		color:yellow;
		background:blue;
		font-size:large;
	}

>上面四种伪类的声明是有一定顺序的，我们简称这种顺序为L-V-H-A。

#### 3.6.5 CSS :focus 伪类

这个伪类适用于已获取焦点的元素的样式，例如：

	input:focus
	{
		color:yellow;
		background:blue;
	}

#### 3.6.6 CSS :first-child 伪类

这个伪类适用于匹配其它元素的第一个子元素，例如：

	p > code:first-child
	{
		color:lime;
		background:red;
	}

语法：

	:first-child
	E:first-child
	E1>E2:first-child

#### 3.6.7 CSS :lang 伪类

这个伪类匹配以语言C表示的元素样式，例如：

	html:lang(zh)
	{
		color:lime;
		background:red;
	}

语法

	:lang(C)
	E:lang(C)

## 4.CSS属性

CSS的属性有很多，这里就记录一些常用的，以后用到再慢慢添加进来。

### 4.1颜色，不透明度和背景属性

opacity是不透明度，0-1的取值；颜色值可以直接填rgb，0-255取值，也可以是百分比取值，也可以是16进制的表示方式。

	color: black;
	color: rgb(50,255,0);
	opacity: 0.5;
	background-color:green;
	background-image:url('html_table.png');
	background-image:none;
	background-image:url('list-orange.png');
	background-repeat:repeat-y;
	background-position:right;
	background-attachment:fixed;

background-repeat -- 定义背景图片的重复方式。取值：

	repeat: 平铺整个页面,左右与上下
	repeat-x: 在x轴上平铺,左右
	repeat-y: 在y轴上平铺,上下
	no-repeat: 图片不重复

background-position -- 定义背景图片的位置，取值:

	left: 左
	center: 中
	right: 右
	top: 上
	center: 中
	bottom: 下
	x% y%
	xpos ypos

background-attachment -- 定义背景图片随滚动轴的移动方式。取值:

	scroll: 随着页面的滚动轴背景图片将移动
	fixed: 随着页面的滚动轴背景图片不会移动

background五个背景属性可以合并在一起定义在background属性中。

### 4.2 文本属性

letter-spacing -- 定义文本中字母的间距(中文为文字的间距)：

	letter-spacing:3px;

word-spacing -- 定义以空格间隔文字的间距(就是空格本身的宽度)：

	word-spacing:30px;

text-decoration -- 定义文本是否有划线以及划线的方式，取值：

	underline: 定义有下划线的文本
	overline: 定义有上划线的文本
	line-through: 定义直线穿过文本
	blink: 定义闪烁的文本

text-transform -- 定义文本的大小写状态,此属性对中文无意义，取值:

	text-align -- 定义文本的对齐方式
	left: 左对齐
	right: 右对齐
	center: 居中
	justify: 对齐每行的文字

text-indent -- 定义文本首行的缩进(在首行文字之前插入指定的长度)：

	text-indent:58%;

white-space -- 空格内元素的显示方式，取值：

	normal: 正常无变化(默认处理方式.文本自动处理换行.假如抵达容器边界内容会转到下一行)
	pre: 保持HTML源代码的空格与换行,等同与pre标签
	nowrap: 强制文本在一行,除非遇到br换行标签
	pre-wrap: 同pre属性,但是遇到超出容器范围的时候会自动换行
	pre-line: 同pre属性,但是遇到连续空格会被看作一个空格

### 4.3 字体属性

font-family -- 定义使用的字体，按顺序获取，如果都没有就使用默认：

	font-family:"宋体",Arial;

font-size -- 定义字体的大小，绝对字体尺寸取值：

	xx-small:最小
	x-small:较小
	small:小
	medium:正常(默认值)，根据字体进行调整
	large:大
	x-large:较大
	xx-large:最大

相对字体尺寸取值：

	larger:相对于父容器中字体尺寸进行相对增大，使用成比例的em作为单位
	smaller:相对于父容器中字体尺寸进行相对减小，使用成比例的em作为单位

font-style -- 定义字体显示的方式:

	normal: 正常
	italic: 斜体
	oblique: 斜体
	inherit: 继承

font-variant -- 定义小型的大写字母字体，对中文没什么意义:

	normal:正常
	small-caps:定义小型的大写字母

font-weight -- 定义字体的粗细,取值:

	normal:正常,等同于 400
	bold:粗体,等同于 700
	bolder:更粗
	lighter:更细

>上面所有属性都可以定在一个font属性中。

### 4.4 列表属性

这个属性是用在列表上的。

list-style-type -- 定义列表样式，取值：

	disc: 点
	circle: 圆圈
	square: 正方形
	decimal: 数字
	decimal-leading-zero: 十进制数，不足两位的补齐前导0，例如: 01, 02, 03, ..., 98, 99
	lower-roman: 小写罗马文字，例如: i, ii, iii, iv, v, ...
	upper-roman: 大写罗马文字，例如: I, II, III, IV, V, ...
	lower-greek: 小写希腊字母，例如: α(alpha), β(beta), γ(gamma), ...
	lower-latin: 小写拉丁文，例如: a, b, c, ... z
	upper-latin: 大写拉丁文，例如: A, B, C, ... Z
	armenian: 亚美尼亚数字
	georgian: 乔治亚数字，例如: an, ban, gan, ..., he, tan, in, in-an, ...
	lower-alpha: 小写拉丁文，例如: a, b, c, ... z
	upper-alpha: 大写拉丁文，例如: A, B, C, ... Z
	none: 无(取消所有的list样式)

list-style-image -- 定义列表样式图片：

	list-style-image:url("/images/list-orange.png");


list-style-position -- 定义列表样式的位置
 
	list-style-position:inside;

>上面列表属性都可以定义在list-style属性上面。

### 4.5 鼠标样式

	body 
	{
		cursor: move;
	}

取值：

	auto: 正常鼠标
	crosshair: 十字鼠标
	default: 默认鼠标
	pointer: 点状鼠标
	move: 移动鼠标
	e-resize,ne-resize,nw-resize,n-resize,se-resize,sw-resize,s-resize,w-resize: 改变大小鼠标
	text: 文字鼠标
	wait: 等待鼠标
	help: 求助鼠标
	progress: 过程鼠标

### 4.6 边框属性

每个内容或元素外面都可以有一个边框，边框分为上边框(top)，下边框(bottom)，左边框(left)，右边框(right)。每种边框有颜色(color)，样式(style)，宽度(width)三种属性。如果上下左右的边框表现不一样，可以分别定义上下左右边框，如果一样可以统一使用border属性定义。

	border-width -- 定义四个边框的宽度
	border-top-width -- 定义上边框的宽度
	border-right-width -- 定义右边框的宽度
	border-bottom-width -- 定义下边框的宽度
	border-left-width -- 定义左边框的宽度

如果只定义三个值，中间的值代表左和右边框的宽度。如果只定义两个值，前面的值代表上下边框宽度，后面的值代表左右边框的宽度。

取值：

	thin: 细
	medium: 中
	thick: 粗

边框颜色：

	border-color -- 定义四个边框的颜色
	border-top-color -- 定义上边框的颜色
	border-right-color -- 定义右边框的颜色
	border-bottom-color -- 定义下边框的颜色
	border-left-color -- 定义左边框的颜色

边框样式：

	border-style -- 定义边框的样式
	border-top-style -- 定义上边框样式
	border-right-style -- 定义右边框样式
	border-bottom-style -- 定义下边框样式
	border-left-style -- 定义左边框的样式

样式的取值：

	none: 无样式
	hidden: 除了同表格的边框发生冲突的时候,其它同none
	dotted: 点划线
	dashed: 虚线
	solid: 实线
	double: 双线,两条线加上中间的空白等于border-width的取值
	groove: 槽状
	ridge: 脊状,和groove相反
	inset: 凹陷
	outset:凸出,和inset相反

border -- 定义四个边的宽度,样式,颜色

	border: border-width border-style border-color;

border不能分别定义4个边框的宽度，颜色和样式，只能统一定义，不可以对四个边设置不同的值，和margin与padding是不同的。

也可以分别来：

	border-top -- 定义上边框样式
	border-right -- 定义右边框样式
	border-bottom -- 定义下边框样式
	border-left -- 定义左边框样式

### 4.7 外白margin属性和内白padding属性

边框的外面可以有一层边外补白(margin)，边外补白可以把块级元素分开。边外补白定义了围绕某种元素(elements)的空白。边外补白只有宽度width一种属性。

	margin -- 定义边外补白
	margin-top -- 定义上边外补白
	margin-right -- 定义右边外补白
	margin-bottom -- 定义下边外补白
	margin-left -- 定义左边外补白

取值：

	<length>: 长度表示法
	<percentage>: 百分比表示法,margin百分比的计算是基于生成的框的包含块的宽度
	auto: 自动

边框的里面可以有一层边内补白(padding)，边内补白定义了边框与边框里面内容的距离。

	padding-top -- 定义上边内补白
	padding-right -- 定义右边内补白
	padding-bottom -- 定义下边内补白
	padding-left -- 定义左边内补白
