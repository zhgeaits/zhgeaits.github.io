---
layout: post
title:  "可扩展标记语言XML学习笔记!"
date:   2011-05-07 11:00:03
categories: xml
type: basic
---

## 1. 什么是XML

xml是EXtensible Markup Language的缩写，即可扩展标记语言，跟html类似，也可以说是基于html慢慢发展过来，这些历史可以从百科上阅读。xml是用来传输数据的，而不是显示数据，这点和html区分很重要，所以，html的每个标签都是已经定义好的了，而xml的标签是没有被预定义的，使用者需要自行定义，也就是说，xml具有自我描述性。

### 1.1 xml语法

所有xml文件的第一行都是这个xml文件的声明，如下所示：

><?xml version="1.0" encoding="utf-8" standalone="yes"?>

version是版本号，即文档符号xml1.0规范；encoding是编码，默认是utf8；standalone是定义文档在一个文件内。

它还有一个样式表处理指令xml-stylesheet，应该是可以用css这些来处理样式的，不过基本没什么用处。

接下来的内容就是一个个的节点，一个xml文件有且仅有一个根节点，节点可以有属性，节点可以包括内容，这些都是很基本简单的语法。另外xml的注释和html是一样的。至于一个xml文档可以使用哪些节点，这个需要dtd来定义。

### 1.2 CDATA

当一个节点包含的内容含有特殊的关键字符时候，需要用CDATA来把整段文本解析为纯字符数据而不是标记。例如，包含大量的<,>,&或者”字符。CDATA节中的所有字符都会被当作元素字符数据的常量部分，而不是XML标记。

><![CDATA[  
>`<a>ggg</a><name>这个标记不会被解析，原样显示</name>`  
>]]>

## 2. 什么是DTD

DTD的意思就是Document Type Definition，就是定义了这个xml文档可以有哪些节点，如果这个xml出现了非法的节点，那么这个xml文档就是非法的。DTD描述了xml文档的结构，其中dtd本质也是一个xml文件，虽然是以dtd作为后缀。

DTD和xml的关系类似于类和对象实例，数据库表和数据库记录的关系。

### 2.1 引用DTD文件

引用dtd文件有两种方式，一是直接在xml文件就定义，而是定义在一个dtd文件，通过外部文件来引用。

1.内部引用，看一个完整的例子：

{% highlight xml %}

<?xml version="1.0"?>
<!DOCTYPE note [
  <!ELEMENT note (to,from,heading,body)>
  <!ELEMENT to      (#PCDATA)>
  <!ELEMENT from    (#PCDATA)>
  <!ELEMENT heading (#PCDATA)>
  <!ELEMENT body    (#PCDATA)>
]>
<note>
  <to>George</to>
  <from>John</from>
  <heading>Reminder</heading>
  <body>Don't forget the meeting!</body>
</note>

{% endhighlight %}

可以看到实际上的语法是：

><!DOCTYPE 根元素 [定义内容]>

其中`根元素`也必须在`定义内容`中被定义的。

2.外部文件引用，语法如下：

><!DOCTYPE根元素 SYSTEM “DTD文件路径”>

看一个mybatis的配置文件例子：

{% highlight xml %}

<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN" 
"http://mybatis.org/dtd/mybatis-3-config.dtd ">

{% endhighlight %}

3.当然是可以内外结合，语法如下：

><!DOCTYPE根元素 SYSTEM “DTD文件路径” [  
>	定义内容  
>]>

一般都是使用外部文件引用的方式较多。

### 2.3定义DTD元素内容语法

**元素的语法：**

><!ELEMENT NAME CONTENT>

其中ELEMENT表示这是一个元素，NAME就是自己定义的节点的名字，CONTENT可以是`EMPTY`，`ANY`，`#PCDATA`。

`EMPTY`的意思是该元素不能包含子元素和文本，但可以有属性（空元素），属性的定义下面有。

`ANY`的意思是该元素可以包含任何在DTD中定义的元素内容。

`#PCDATA`的意思是可以包含任何字符数据，但是不能在其中包含任何子元素。

另外CONTENT还可以是组合类型，即content可以是指定的元素和相应的符号，例如：

><!ELEMENT 家庭 (人+, 家电*)>

其中，人和家电分别是另外定义的元素，上面的定义意思是家庭至少包含一个人，和0或者多个家电，并且按顺序出现；关于符号的描述如下：

![xml xml](/image/xml.png "xml")

**元素属性的语法：**

><!ATTLIST 元素名称 属性名称1 属性类型1 属性特点1 属性名称2 属性类型2 属性特点2>

下面是属性类型和属性特点的取值：

![xml xml](/image/xml2.png "xml")

## 3. 什么是Schema

对于上述的DTD，它有很多的局限性，例如不遵循xml语法，数据类型有限，不可扩展，不支持命名空间，所以schema是为了改善dtd的。所以现在基本上都是使用schema的了，简称为xsd，xml schema definition。

### 3.1 命名空间xmlns

为什么需要使用命名空间，假设两个xml文档，都有元素table，那么当同时读取这两个文档的时候，就会发生冲突了。所以要使用命名空间，例如下面这个xml文件这样编写：

{% highlight xml %}

<f:table xmlns:f="http://www.w3school.com.cn/furniture">
   <f:name>African Coffee Table</f:name>
   <f:width>80</f:width>
   <f:length>120</f:length>
</f:table>

{% endhighlight %}

语法很简单，就是xmlns:prefix="url"，实际上这个url并没有实际意义，只有用上了schema才有意义的。如果没有用上前缀，那么就是定义了默认的命名空间。另外，命名空间除了可以用在节点元素上，也是可以用在节点的属性上的。

### 3.2 xsd的引用

xsd是代替了dtd的使用的，首先我们看看怎么使用它，显然在xml文件第二行开始是没有了<!DOCTYPE>的了，直接就是第一个根节点，而xsd实际上是通过xmlns来引用的，例如spring的配置文件：

{% highlight xml %}

<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:aop="http://www.springframework.org/schema/aop"
	xsi:schemaLocation="http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
						http://www.springframework.org/schema/aop/spring-aop-2.5.xsd">
	<aop:config>
	</aop:config>
</beans>

{% endhighlight %}

可以看到有一个默认命名空间和xsi和aop的命名空间，那么beans的定义实际上是出自默认的命名空间的，所以没有前缀。aop是spring的切面编程配置。xsi是schema的规范命名空间，定义这个就可以使用schemaLocation属性了，这个属性是用来指定xsd的文件路径。这个时候的命名空间url就有了实际意义了。于是beans的子节点就可以使用aop这个命名空间了，例如节点<aop:config>。

注意到为什么xsi没有指定xsd？因为这个是官方默认规范地址，可以认为是永久不变的，不需要指定路径。

### 3.3 关于XSD

说到xsd是代替了dtd，解决了它的局限性，实际上xsd也是一个xml文件，只不过它不是xml后缀而已。这似乎有点递归，有点像编程语言一样，xml由xsd定义，xsd其实又是xml文件来的，那xsd由上面来定义呢？其实是命名空间。且看spring的这个aop的xsd：

{% highlight xml %}

<xsd:schema xmlns="http://www.springframework.org/schema/aop" 
			xmlns:xsd="http://www.w3.org/2001/XMLSchema" 
			targetNamespace="http://www.springframework.org/schema/aop" 
			elementFormDefault="qualified" attributeFormDefault="unqualified">
  <xsd:import namespace="http://www.w3.org/XML/1998/namespace"/>

</xsd:schema>

{% endhighlight %}

首先看到的是使用了xsd的命名空间，指向了schema的规范地址，所以xsd:schema这个节点出自那里，这个节点有一个targetNamespace，就是定义了这个xsd的命名空间了，它和这个xsd文件的默认命名空间一样了，并没有关系。elementFormDefault="qualified"说明他定义的元素都必须带有命名空间的。attributeFormDefault="unqualified"说明他定义的属性不一定需要带有命名空间的。xsd:import是引入了其他的命名空间，非常易于扩展使用。

和上面一样，schema这个节点虽然知道是出自xsd这个命名空间，但是并不知道定义它的地方在那里，还有没有需要指定一个xsd地址？岂不死循环了。其实这个是官方规范，应该是官方指定的，默认地址永久不变的了，里面有各种官方的定义了。

### 3.2 schema语法

xsd要学习的东西还是蛮多的，这里就只学习了简单的一些语法，以后如果涉及到要编写xsd的话，再去详细看它的语法，非常适合去看spring这些框架的xsd来学习。且看一个简单的定义：

{% highlight xml %}

<xs:element name="lastname" type="xs:string"/>
<xs:element name="age" type="xs:integer"/>
<xs:element name="dateborn" type="xs:date"/>
<xs:element name="color" type="xs:string" default="red"/>
<xs:element name="employee">
  <xs:complexType>
    <xs:sequence>
      <xs:element name="firstname" type="xs:string"/>
      <xs:element name="lastname" type="xs:string"/>
    </xs:sequence>
  </xs:complexType>
</xs:element>

{% endhighlight %}

## 4. 用java解析XML

上面学习了xml以后，用什么语言进行解析xml文档就是简单的事情了，只不过是使用不懂的工具罢了，不过还是主要分成两种类型，一是基于事件驱动的解析，一种是基于DOM节点树的解析。主要区别是，前者不需要加载整个文档就可以解析，但是却不能像后者一样灵活随机访问了。

常用的解析工具有JAXP，JDOM，DOM4J等，都是简单的东西，如果用到以后，只要去官网看一下文档就能学习使用了。

## 5. 用JS解析XML

对于JS解析xml，基本上浏览器都是内置了对象可以解析的了，使用常用的框架就能调用了，例如JQuery。

## 6. android/ios上使用xml

后来学习android以后的补充：

在android当做，基本上所有的配置和界面都是使用xml来实现的了，但是它不像web那样使用标准化的xml，毕竟，这里android是用来编写界面显示数据了。首先解析xml是android系统的事情了，它没有具体指定使用哪些xsd，基本和在开发的时候，ide解析xml，所有节点都是合法的，只不过不能解析成为合法的界面组件而已。然后每个布局并没有使用默认的命名空间，而是使用一个android的命名空间，如下：

>xmlns:android="http://schemas.android.com/apk/res/android"

也没有使用xsi:schemaLocation来指定xsd的url，这就说明了，其实xsd是android系统或者sdk内置的了。当我们在values下的配置文件中输入一个节点的时候，ide会提示补全我们可以输入哪些合法节点和属性，这就是sdk内置了xsd来定义节点的，配合sdk在ide中使用。如果在布局文件里面，我们是可以使用任何自定义的UI组件的，所以，输入任何节点都是合法的，不过是解析不合法而已。任何一个节点的属性必须使用android前缀，这就是android这个命名空间的作用，使用了官方的UI属性。

如果是自定义了组件采用这个命名空间来使用自定义的属性：

>xmlns:zg="http://schemas.android.com/apk/res-auto"

不管多少个自定义组件，都是使用这个命名空间的了，它是由系统自动解析attrs.xml下的自定义属性，然后生成对应的xsd，然后对应于这个res-auto的命名空间了。