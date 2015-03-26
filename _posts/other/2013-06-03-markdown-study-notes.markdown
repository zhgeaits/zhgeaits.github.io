---
layout: post
title:  "编写这个blog的轻量级标记语言markdown的一些备忘使用记录"
date:   2013-06-03 11:00:03
categories: other
type: other
---

这些blog全都是用markdown来写的，jekyll支持解析markdown，也支持解析html，但是markdown更容易编写，为什么要学习markdown？就像写wiki一样，也要记录很多语法！
反正，学了markdown，别人会觉得你很高大上！

1.按两个空格键然后回车就可以实现换行了

2.>是标记段落开始

3.有些符号要转义，如#*._{}[]()+-!使用转义符号\

4.使用图片：  
<pre>
行内形式（title 是选择性的）：
![alt text](/path/to/img.jpg "Title")

另外一种形式：
![alt text][id]
[id]: /path/to/img.jpg "Title"
</pre>

5.使用超链接：
<pre>
行内形式是直接在后面用括号直接接上链接(title可选)：
This is an [example link](http://example.com/ "With a Title").

参考形式的链接让你可以为链接定一个名称，之后你可以在文件的其他地方定义该链接的内容：
I get 10 times more traffic from [Google][1] than from
[Yahoo][2] or [MSN][3].

[1]: http://google.com/ "Google"
[2]: http://search.yahoo.com/ "Yahoo Search"
[3]: http://search.msn.com/ "MSN Search"
</pre>

6.语法高亮：

这里{和%之间多了一个空格。
<pre>
{ % highlight xml % }
{ % endhighlight % }
</pre>

7.区域：

	<pre>
	</pre>
	
8.代码区域内的文字不会被处理，按照原样输出。每一行前边加入4个空格或者一个tab可以标记一个代码段落