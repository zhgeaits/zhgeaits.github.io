---
layout: post
title: "JavaWeb服务器页面JSP学习记录"
date: 2011-03-26 01:15:24
categories: javaweb
type: java&web
---

## 1 什么是JSP

学习过servlet以后知道，如果servlet直接返回一些数据那还是比较好的，但是如果返回一个页面的话就很蛋疼了，必须写html的代码，这些页面的逻辑不应该耦合在这里。所以java就设计了jsp，目的就是要分离页面，让servlet只要处理业务逻辑就好了。JSP的意思是java server page，其实就是嵌套了html/css/js和java的代码的一个页面，实际上最终经过编译以后还是转换成了servlet来的，虽然jsp文件会转换成servlet的类，但是不会去编译成class，只有在首次访问的时候才会去编译。因此学会了servlet就是学会了一半jsp了，具体jsp怎么变成servlet的，也就是request的访问流程是怎么处理的，这个必须要了解javaee的规范了，有兴趣再去深入学习了。既然jsp是一个页面，那么jsp其实是可以直接通过url来访问的，就不用像servlet那样在web.xml配置了。

>源码地址看servlet那篇blog

## 2 配置使用

直接在MyEclipse里面，项目的WebRoot目录下新建一个jsp文件即可，这样就会有一个简单的模板生成，方便很多。如果在WebRoot目录下新建一个目录，那么相应的访问jsp的url也要多一级。所以使用jsp的地方大概有三处地方：

1.在web.xml那里配置welcome-file的时候，一般设置index.jsp。

2.直接跳转到某个url的时候指定一个jsp后缀的文件。

3.在servlet执行完毕以后请求转发到某一个jsp去。

前两种一般我们都不建议使用，不希望用户直接访问jsp文件，我们会使用filter来过滤这样的url，而换成.do或者.action的url，所以基本上都是第三种方法的实现，因此在转发的时候就会带上数据，然后jsp里面可以访问request和response对象，进而显示数据。

## 3 JSP对象与元素

jsp大部分功能都是展示html页面，如果html那三个都懂了，基本上没什么难的了，只需要学习一些jsp的元素来操作数据即可。元素有三种：脚本元素，指令元素和动作元素，现在动作元素已经不用了，因为很多指令元素已经可以代替它的功能了。

### 3.1 隐藏对象

先来看看有哪些对象常用的。

1.输入输出读取数据的对象：request, response, out

2.作用域通信对象：session, application, pageContext

3.Servlet对象：page, config

4.异常对象：exception

### 3.2 脚本元素

采用<%%>包围起来的，里面可以是表达式，或者java代码，或者是声明变量或者方法。

表达式：

><%=a%> == <%out.print(a);%>

Java代码：

	<%
	String path = request.getContextPath();
	String basePath = request.getScheme()+"://"+request.getServerName()+":"+request.getServerPort()+path+"/";
	%>

声明：

	<%! %>声明变量时全局的  
	<% %>声明变量是局部的

### 3.3 指令元素

指令是以<%@ %>符号包围的，语法是`<%@ 指令名称 属性1="属性值1" 属性2="属性值2" %>`，指令有三种，分别是page，include和taglib。

#### 3.3.1 page指令

在JSP中的任何地方、以任何顺序，都可以包含任意数量的page指令。我们新建一个jsp文件的时候，模板上面默认就有一条page指令。它常用有这几个属性：

	language：定义要使用的脚本语言
	import：将包和类导入
	buffer：设置用来存储客户端请求的缓冲区的大小
	errorPage：定义处理异常的JSP 页面
	isErrorPage：表示当前页面能否作为错误页面
	contentType：指定页面格式和所使用的字符集
	isELIgnored：是否使用EL表达式
	isThreadSafe：是否线程安全

**EL表达式**

page指令的EL表达式可以很方便的获取变量的值，进而代替了脚本元素的表达式方式，例如脚本需要拿到request对象然后get参数。使用EL的话直接点参数的key就可以了。声明使用EL：`<%@ page isELIgnored="false"%>`。

><%=request.getAttribute("username")%>和${requestScope.username}是一样的意思

EL可以访问的对象有：pageContext，pageScope，requestScope，sessionScope，applicationScope。

各个对象能用到的时候再来补充。

既然EL是表达式，那么肯定有运算符，并且可以使用三目表达式，例如：

>${(empty requestScope.name)? 'name is null':requestScope.name}

运算符：

	==或者eq
	!=或者ne
	<或者lt
	>或者gt
	<=或者le
	>=或者ge

#### 3.3.2 include指令

include 指令用于在运行时将HTML文件或JSP页面嵌入到另一个JSP页面，语法是`<%@ include file = "文件名"%>`，所以页面不要定义相同名称变量，page指令除去import属性，其它属性值要一致。

#### 3.3.3 taglib指令

这个指令允许在JSP页面使用自定义的标签，语法是`<%@ taglib uri="标签库的url" prefix="c"%>`。

最常用的就是jstl这个标签库了，结合EL来使用基本上能完成所有的功能了：

><%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c"%>

然后就可以用<c:XX这样的标签来使用了，以后用到就来这里补充：

if标签：

	<c:if test="${empty requestScope.name}" var="result" scope="page"> name is null;</c:if>

forEach标签：

	<c:forEach begin="2" end="10" varStatus="status" step="2">
		index=${status.index } ++ count=${status.count }
	</c:forEach>

另外还有I18N格式标签库，SQL标签库，XML标签库和函数标签库等等。