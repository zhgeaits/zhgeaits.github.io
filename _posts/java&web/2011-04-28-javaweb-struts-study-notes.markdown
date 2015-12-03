---
layout: post
title: "经典的MVC设计模式产品Struts学习记录"
date: 2011-04-28 01:15:24
categories: javaweb
type: java&web
---

## 1 什么是Struts

学完servlet和jsp以后，又有了数据库，jdbc，前端的知识基础，开发一个网站产品是一件很容易的事情了，所以我也因此练习了两个项目，一个xaygc的卖书商店，一个是企业信息管理系统，项目在TrainingForWeb；虽然项目很小，但是各方面知识都具备了，还有DAO的框架（见JDBC的blog），很值得学习的。

但是开发起来还是有很多不便的，没有实现低耦合高内聚的设计原则，也就是不太方便以后的扩展与维护。于是，伟大的apache又开发了一个顶级项目struts，基于mvc设计模式的；但是struts并不是对于开发来说很便捷，所以别的公司还在开发更好的框架，webwork就是struts2的前身，后面被apache收购了，它是用拦截器的机制来处理用户的请求，这样的设计也使得业务逻辑控制器能够与ServletAPI完全脱离开，对于开发来说更加便利了。

## 2 MVC设计模式

jsp刚出来的时候，因为里面即可以写java，又可以写前端代码，就像php一样，所以基本上所有的代码都是在那里写的，访问数据库也是在那里，这样实在是太复杂了，于是java就出了model的规范，如下图所示：

![alt text](/image/model1.jpg "model1")

JSP页面负责表现逻辑、控制逻辑，JavaBean负责业务实现、持久化逻辑；这样对部分业务逻辑的进行了封装，但是缺乏对控制逻辑的封装，jsp及负责表现逻辑，又负责控制逻辑。于是又出现了model2的规范，如下图所示：

![alt text](/image/model2.jpg "model2")

JSP页面仅负责表现逻辑，JavaBean负责业务实现、持久化逻辑，Servlet负责流程控制，彻底分离了业务逻辑与表现逻辑，进一步简化了JSP页面，但是Servlet往往难以维护，就算使用那个DAO框架，所以才会出现struts那些框架。

其实上面model2就是mvc框架的一个比较好的实现了，但是还不算完全。mvc是模型(model)，视图(view)和控制器(controller)的缩写，把业务逻辑、数据和界面显示进行了分离，将业务逻辑聚集到一个部件里面，在改进和个性化定制界面及用户交互的同时，不需要重新编写业务逻辑。如下图所示：

![alt text](/image/mvc.png "mvc")

Model是应用程序中用于处理应用程序数据逻辑的部分，通常模型对象负责在数据库中存取数据。例如使用的DAO框架就是一个Model，包括了bean和操作数据的逻辑。

View是应用程序中处理数据显示的部分，通常视图是依据模型数据创建的。例如jsp现实页面数据。

Controller是应用程序中处理用户交互的部分，通常控制器负责从视图读取数据，控制用户输入，并向模型发送数据。例如servlet。

## 3 struts1最经典的第一个版本

官方网站是：http://struts.apache.org/。有了struts以后，真的开发会方便很多，例如拦截，校验，表单，标签等等功能。去官网下载完整版本，包括examples, lib，source和doc，里面有详细的教程，也方便学习。

### 3.1 配置使用

再怎么厉害，复杂，牛逼设计的框架，也只不过是一个工具而已，从高层次来看，它就是一个软件，和平时我们下载一个软件差不多，都是开发人员写出来给别人使用的；所以，我们先不需要了解它的内部，第一步就尽管使用它就好了。参考examples里面struts-blank-1.3.10，看看这个空白项目里面添加了哪些支持。

先忽略maven和Ant的配置构建文件，发现这个blank有以下和别的web项目不一样的（主要是在WEB-INF下）：

1.在src/java目录下有这个文件MessageResources.properties，其实这个文件是放在src目录的包里面即可的。这文件是配置一些变量的地方，主要是一些错误码等等。

2.libs目录下添加了支持struts需要的jar包。

3.添加了一个validation.xml文件，这个是配置了struts的字段校验功能，并非必须的。

4.添加了struts-config.xml文件，这个是struts的主要配置文件，重点！

5.web.xml文件里面添加了ActionServlet这个servlet，mapping的url是*.do。主要是配置读取struts-config.xml。

**MyEclipse是可以直接配置struts支持的**

新建一个web项目以后，右键选择MyEclipse，再选择Add Struts Capabilities就可以选择添加struts各个版本的支持。最后发现改变的东西和blank项目几乎一样。

### 3.2 struts1的MVC流程

<img src="/image/struts1_mvc.jpg" width="700px">

因为在web.xml配置了ActionServlet，所以所有的*.do请求都会来到这个servlet，它只是一个中央控制器，并不是MVC里面的controller。ActionServlet读取了struts-config.xml配置，这个配置文件主要是配置了formbean和action。Action就是controller，它处理主要的业务逻辑，formbean是一个model存储了各种数据，实际就是一个javabean，是struts通过反射生成的，而视图view是jsp文件。ActionServlet根据请求的url分发给相应配置的action进行处理，action返回跳转的url，实际上交给了ActionServlet来处理跳转响应浏览器的。

### 3.3 一个登陆例子

1.在struts-config.xml配置formbean和action，如下：

{% highlight xml %}

<form-beans>
	<form-bean name="loginForm"    
    type="com.zhangge.struts.LoginForm">
	</form-bean>
</form-beans>

<action-mappings>
	<action path="/login" name="loginForm"
		type="com.zhangge.struts.LoginAction">
		<forward name="ok" path="/ok.jsp"></forward>
        <forward name="error" path="/error.jsp" redirect="true"></forward>
	</action>
</action-mappings>

{% endhighlight %}

ActionServlet首先根据url请求/login.do找到action-mappings节点下的某个action节点的path属性，path的属性值必须是唯一的，然后判断此action节点是否有name属性，如有，找到form-beans节点下某个form-bean节点，创建type指定的formBean类。

2.LoginForm和LoginAction的实现。

这个就不详细了，formBean必须继承actionForm，是标准的JavaBean，属性名必须与客户端请求参数名一致，前三个字母要小写。action必须继承Action，需要实现excute方法。

3.页面请求和跳转

注意到action配置的path是带有斜杠/的，在jsp编写表单如下：

{% highlight xml %}

<form id="loginForm" action="${pageContext.request.contextPath}/login.do">
	<input type="text" name="name"/>
	<br>
	<input type="password" name="password"/>
	<br>
	<input type="submit" name="login" value="login"/>
</form>

{% endhighlight %}

action在执行完excute方法以后调用ActionMapping.findForward()方法返回一个ActionForward进行跳转，方法传递的就是一个String，struts会根据这个string去struts-config配置action节点下forward节点的name属性进行匹配，当对上了以后就会掉转到响应的jsp。

至于参数传递，在excute方法里面带有formbean，request和response参数了，一般我们直接使用formbean就足够了，也可以从request里面获得参数。所以在request里面set参数以后，在jsp也就获取得到了。

### 3.4 校验Validation

一般开发都会需要用到校验的功能。这里就不说明了，参考文档或者sample即可。

## 4 struts2最实用的第二个版本