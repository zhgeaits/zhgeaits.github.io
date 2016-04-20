---
layout: post
title: "经典的MVC设计模式产品Struts"
date: 2011-04-28 01:15:24
categories: javaweb
supertype: career
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

{% highlight html %}

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

一般开发都会需要用到校验的功能。这里就不说明了，参考文档或者sample即可。还会用到很多标签库之类.....struts的更多使用可以看文档，也可以直接google别人怎么使用的，最好还是看pdf的书，人家分析比较清晰那些设计思想的...

## 4 struts2 最实用的第二个版本

官方网站依旧是：http://struts.apache.org/。去下载最新的稳定版本然后解压，发现里面也是apps，docs，libs和src，包含了详细的教程。

相比struts1，struts2的确好很多，更适合开发

### 4.1 配置使用

和struts1一样的学习过程，直接看struts2-blank，还是看看WEB-INF目录多了些什么东西。

1.libs目录下添加了支持struts需要的jar包。

2.src/java目录下包含了struts.xml，example.xml，log4j2.xml，Login-validation.xml和velocity.properties这几个文件。log4j2.xml是日志文件，可以先不关注。velocity.properties是另外一个技术，也先不用关注。而struts.xml是主配置文件，是重点。example.xml只是struts.xml的一个子文件，最终是在struts.xml里面通过include包含进去的。Login-validation.xml是校验配置，很简单的。

3.web.xml里面配置了StrutsPrepareAndExecuteFilter这个filter，不再是servlet了，而filter的mappingurl是/*。security那些节点的配置暂时不知道干嘛用的可以先不管。

然后把这些配置复制到自己的项目即可了。

**MyEclipse是可以直接配置struts支持的**

新建一个web项目以后，右键选择MyEclipse，再选择Add Struts Capabilities就可以选择添加struts各个版本的支持。最后发现改变的东西和blank项目几乎一样。

### 4.2 struts2的原理流程

![alt text](/image/struts2_flow.png "mvc")

struts2也是mvc模式的实践，Action是一个控制器Controller，JSP页面是视图View，后面的JavaBean是模型Model。

![alt text](/image/struts2_core.png "mvc")

第一个图还是比较抽象的，第二个图才是比较详细的架构。相比struts1，struts2采用了大量的filter和interceptor来实现的（Servlet在哪里？哪里封装了它？）。主要是FilterDispatcher被调用，然后调用ActionMapper来决定这个请是否需要调用某个Action，如果ActionMapper决定需要调用某个Action，FilterDispatcher把请求的处理交给ActionProxy，ActionProxy通过ConfigurationManager读取框架的配置文件，找到需要调用的Action类。然后ActionProxy会创建一个ActionInvocation的实例，ActionInvocation实例使用命名模式来调用，在调用Action的过程前后，涉及到相关拦截器（Intercepter）的调用。最后Action执行完毕，通过result返回到不同的页面上。

### 4.3 struts2对比struts1

1.可以直接知道的是struts2看不到哪里有servlet了，主要框架使用filter来实现，里面包含了大量的拦截器，这样的框架可以让我们做更多的业务，如权限控制等等。

2.Action明显没有依赖servlet的api了，这样非常方便测试，不需要初始化那些request，response对象等等，也就是说测试action根本不用考虑web容器！

3.Action没有依赖框架，也就是没有明显的侵入式设计，只是继承ActionSupport。于是在重构的时候，Action是非常方便移植和重用的。

4.Action每次请求都是一个新实例对象，这是线程安全的。

5.其他细节的不同就不再对比了。

### 4.4 一个登陆例子

同样是一个登陆例子，其实也就是struts2-blank里面的代码而已。

1.web.xml已经配置了一个filter以后就不需要过多配置什么了，只要关注struts.xml就可以了。

{% highlight xml %}
<struts>
	<constant name="struts.enable.DynamicMethodInvocation" value="false" />
	<constant name="struts.devMode" value="true" />
	
	<package name="default" namespace="/" extends="struts-default">
	
	    <default-action-ref name="index" />
	
	    <global-results>
	        <result name="error">/WEB-INF/jsp/error.jsp</result>
	    </global-results>
	
	    <global-exception-mappings>
	        <exception-mapping exception="java.lang.Exception" result="error"/>
	    </global-exception-mappings>
	
	    <action name="index">
	        <result type="redirectAction">
	            <param name="actionName">HelloWorld</param>
	            <param name="namespace">/example</param>
	        </result>
	    </action>
	</package>
	
	<package name="example" namespace="/example" extends="default">
	
	    <action name="HelloWorld" class="example.HelloWorld">
	        <result>/WEB-INF/jsp/example/HelloWorld.jsp</result>
	    </action>
	
	    <action name="Login_*" method="{1}" class="example.Login">
	        <result name="input">/WEB-INF/jsp/example/Login.jsp</result>
	        <result type="redirectAction">Menu</result>
	    </action>
	
	    <action name="*" class="example.ExampleSupport">
	        <result>/WEB-INF/jsp/example/{1}.jsp</result>
	    </action>
	
	    <!-- Add actions here -->
	</package>
</struts>
{% endhighlight %}

可以看到根节点是struts，下面节点主要是package和一些常量配置，而且package是可以继承的！package下的主要节点是action，action下的主要节点是result。package的主要属性是namespace，namespace则决定了action的访问路径。注意到default这个package，里面配置的默认访问路径是/，而默认访问的action是index，还配置了全局的result和全局的exception。而example这个package则是配置了3个action。

2.编写一个LoginAction

{% highlight java %}
public class Login extends ActionSupport {

    @Override
    public String execute() throws Exception {
        return SUCCESS;
    }

    @SkipValidation
    public String form() throws Exception {
        return INPUT;
    }

    private String username;

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    private String password;

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }
}

{% endhighlight %}

这个action是继承ActionSupport的，但是这并不是必须的。还可以发现，参数都是通过set方法注入的，并且命名方面还是和struts1一样；而get方法是用于在在页面上读取的数据，还可以返回对象等等。默认是执行execute()方法的，而返回的则是string，struts根据返回的string找到相应的result进行响应。

3.struts.xml里面一个比较重要的特性就是通配符的，然后通过配置method属性可以指定执行的方法。一个*就是一个通配符，然后用{1}来读取这个值，还可以应用到result里面去。观察上面的Login_*这个action即可明白了。

4.action底下的是result这个节点，它有四种类型type，分别是dispatcher，redirect，chain，redirectAction；默认是dispatcher这个类型，具体各个区别最好还是需要看看文档，dispatcher应该是带有参数回去的，redirect是重定向到新的URL，result里面填写的内容都是指定jsp；redirectAction则是跳到另外一个Action上去，result里面的内容则是action的名字。所以一般使用这三个就足够了。

到这里就完成了一个简单的例子，不需要写formbean这个鬼东西，使用ModelDriven的设计模式，struts2开发更方便，只需要编写action就完了，action还很方便junit来测试！

### 4.5 其他的技术

struts还有更多的技术，标签库，拦截器，国际化，异常处理；理解值栈等等这些还是需要好好深入去学习的。

>PS：以上配图均来自互联网。