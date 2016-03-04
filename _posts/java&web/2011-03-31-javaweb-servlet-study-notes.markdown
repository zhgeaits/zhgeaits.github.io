---
layout: post
title: "JavaWeb服务程序Servlet组件"
date: 2011-03-31 01:15:24
categories: javaweb
type: java&web
---

## 1 什么是servlet

JavaEE是一套很大很全的技术规范，详细可以google更多标准文档。而javaweb只不过是能响应网络请求的服务器，servlet就是这样的一个小程序，它的名字是server+applet的合称，而applet你懂得！虽然servlet小，但是它却是核心，http的请求是交给它来处理响应的，当然前后会经过很多处理。而tomcat上面运行的就是servlet程序。

>javaee都是开源的，下载源码的地址是：https://java.net/projects/servlet-spec/。但是这个地址一般没有svn的权限，下载不了，可以来这里：http://grepcode.com/project/repo1.maven.org/maven2/javax.servlet/javax.servlet-api/。其实很多开源的maven库都是可以去拿到源码的。这个是javaee的api：http://grepcode.com/snapshot/repo1.maven.org/maven2/javax/javaee-api/7.0

## 2 Servlet的API和生命周期

### 2.1 API

Servlet是一个interface，主要定义了三个方法init(), service()和destory()。继承Servlet的有一个抽象类GenericServlet，实现了一些与协议无法的接口。然后就是实现了http协议的HttpServlet类，继承自GenericServlet，也是抽象类。一般我们写一个servlet就是继承HttpServlet，然后重写doGet和doPost方法。当然，我们应该了解更多servlet的实现，最好去看看HttpServlet这个类，我们就会比较清楚一个请求的流程了。

### 2.2 生命周期

Tomcat服务器启动的时候就会实例化所有配置的servlet了，然后会调用servlet的init()方法。当有一个请求进来的时候，找到相应的servlet，然后去调用service()方法，而HttpServlet实际上处理了这个请求，会相应调用doGet或者doPost方法的。最后如果tomcat关闭了就会相应调用destory()方法的。

### 2.3 接口方法

servletConfig对象是对于下面的配置的。

	servletConfig = super.getServletConfig()
	servletConfig.getInitParameter("userName")：得到参数值
	servletConfig.getServletName() ：得到servletName
	servletConfig.getServletContext():得到ServletContext类的实例application对象

application对象的生命周期对应于一个web应用。

## 3 配置

在MyEclipse中创建一个web项目以后，在项目的WebRoot\WEB-INF目录下有一个web.xml文件，里面配置了所有的servlet，可以打开一个tomcat的webapps目录examples里面的例子参考，那是比较全面的，下面是我的一个例子：

{% highlight xml %}

<?xml version="1.0" encoding="UTF-8"?>
<web-app version="2.5" 
	xmlns="http://java.sun.com/xml/ns/javaee" 
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
	xsi:schemaLocation="http://java.sun.com/xml/ns/javaee 
	http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd">

<servlet>  
    <description>xhome</description>  
    <display-name>HelloWorld_xhome </display-name>  
    <servlet-name>HelloWorld</servlet-name>  
    <servlet-class>org.xhome.servlet.HelloWorldServlet</servlet-class>  
    <init-param>  
        <param-name>driver</param-name>  
        <param-value>aaaaaa-8</param-value>  
    </init-param>  
    <init-param>  
        <param-name>url</param-name>  
        <param-value>127.0.0.1</param-value>  
    </init-param>  
    <load-on-startup>3</load-on-startup>  
</servlet>  
  
<servlet-mapping>  
    <servlet-name>HelloWorld</servlet-name>  
    <url-pattern>/hello</url-pattern>  
</servlet-mapping>  
  
<servlet-mapping>  
    <servlet-name>HelloWorld</servlet-name>  
    <url-pattern>/world</url-pattern>  
</servlet-mapping>

<welcome-file-list>
    <welcome-file>index.jsp</welcome-file>
</welcome-file-list>

<context-param>
    <param-name>context/param</param-name>
    <param-value>avalible during application</param-value>
</context-param>

</web-app>

{% endhighlight %}

说明如下：

	<description><display-name>是对servlet的描述
	<servlet-name><servlet-class>是指定servlet的名字和位置
	<init-param>是配置初始化参数，在servlet中可以获得。servletConfig.getInitParameter("driver ")。
	<servlet-mapping>是对一个servlet节点的映射，<servlet-name>要与servlet节点的名字对应，<url-pattern>是在浏览器的地址栏输入的地址
	<welcome-file-list>是服务器打开应用的第一个执行的欢迎文件
	<context-param>是定义application范围内的参数，存放在servletcontext中

## 4 Request请求和Response响应

Request表示请求，就是java封装的一个类而已，其最重要的一个功能是封装客户端请求参数，重点是理解http协议就明白什么是请求了，我们可以通过request的一些方法得到这些请求参数的数值。要深入学习还得好好去看看java是如何解析web的http协议请求，然后做处理，最后封装成request类的。

当servlet处理完业务以后，需要把数据传回客户端，或者返回客户端一个html页面，其实返回jsp也就是返回html页面，然后这些封装成为一个response对象，最后交给了tomcat处理。


### 4.1 Request的其它重要方法
 
	request.getParameter("pName");获得单个数值的请求的值
	request.getContextPath()：得到web应用程序名称 
	request.getServletPath()：如果请求的是servlet，那么返回的该servlet的url-pattern，如果是jsp，返回jsp文件名。 
	request.getRequestURI() = request.getContextPath() + request.getServletPath() 
	request.getRequestURL()：请求的全部路径，绝对路径 
	request.getMethod()：请求类别 
	request.getRemotePort()：客户端请求端口 
	request.getRemoteAddr()：得到客户端的IP地址 
	request.getRealPath(“/”):得到当前Web应用程序的绝对路径 
	request.getHeader("User-Agent"):得到客户端浏览器的信息 
	request.getRequestDispatcher(“<url-pattern>”):得到请求转发对象 

### 4.2 request中parameter与attribute的区别

	parameter是客户端传过来的数据，attribute是在Server端放入的数据。
	parameter数据是只读的，attribute是可读写的。
	parameter相关的方法有request.getParamenter();
	attribute相关的方法有setAttribute(String,Object)，getAttribute(String)，removeAttribute(String) 
	getParameter返回的是String类型，而getAttribute返回的是Object类型，需要强转。

### 4.3 请求转发与重定向

现在我们开发的程序大多客户端发来了请求，由一个Servlet处理完之后马上给客户端应答，但真正的开发中，一个Servlet往往处理不了所有业务，我们需要多个Servlet协同工作处理，那么这里就有两个问题：第一，如何由一个Servlet到另外一个Servlet;第二，这两个Servlet如何传递数据，或者说两个Servlet之间如何通信。由此，我们引出在开发web应用时非常重要的两个概念：请求转发与重定向。

请求转发特点：可以传递request中的数据（包含parameter与attribute两方面数据），或者说request中的数据不会丢失；只能在一个webApp内转发。示例代码如下：

{% highlight java %}

public void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException { 
	System.out.println("得到客户端的请求参数:"); 
	System.out.println("name = " + request.getParameter("name"));
	request.setAttribute("zhangge", "shuaige"); 
	RequestDispatcher rd = request.getRequestDispatcher("/nextservlet1"); 
	rd.forward(request, response); 
}

{% endhighlight %}

然后再另一个servlet/jsp里面的代码如下：

{% highlight java %}

public void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException { 
	System.out.println("还能得到客户端的请求参数:"); 
	System.out.println("name = " + request.getParameter("name"));;
	System.out.println("zhangge = " + request.getParameter("zhangge"));
	RequestDispatcher rd = request.getRequestDispatcher("/nextservlet2"); 
	rd.forward(request, response); 
}

{% endhighlight %}

重定向特点：不能传递request中的数据（包含parameter与attribute两方面数据），或者说request中的数据会丢失；重定向可以定向到其它应用程序。示例代码如下：

{% highlight java %}

public void doPost(HttpServletRequest request, HttpServletResponse response)  throws ServletException, IOException { 
	System.out.println("得到客户端的请求参数:");
	System.out.println("name = " + request.getParameter("name")); 
	request.setAttribute("zhangge", "shuaige");
	response.sendRedirect("/webapp2/servlet1"); 
}

{% endhighlight %}

实际上，请求转发是服务器内部的跳转，从一个servlet跳转到另外一个servlet；而重定向则是服务器外部的跳转，一个servlet相应了客户端，客户端重新跳转到新的url上去，所以重定向调用的response的方法。

### 4.4 Cookies和Session

关于这两个的概念情况另外一篇blog。

{% highlight java %}

//设置生命周期
Cookie cookie=new Cookie("user",user);
cookie,setMaxAge(day*24*60*60);
response.addCookie(cookie);

//获取cookie值
Cookie[] cookies=request.getCookies();
if(cookies!=null){
 	for(Cookie cookie:cookies){
		if(cookie.getName().equalsIgnoreCase("user"){
			user=cookie.getValue():
			break;
		}
	}
}

//注销cookie，在servlet里面
Cookie[] cookies=request.getCookies();
for(Cookie cookie:cookies){
    if(cookie.getName().equals("user")){
		cookie.setValue(null);
		//Cookie cookie2=new Cookie(cookie.getName(),null);
		//cookie.setMaxAge(0);
	}
	response.addCookie(cookie);
}

{% endhighlight %}

## 5 Listener监听器

这个监听器说的是在application，session，request三个对象创建消亡或者往其中添加修改删除属性时触发的动作。我们只要实现了相应的listener接口，然后在web.xml配置即可。配置如下：

{% highlight xml %}

<listener>	
	<listener-class>org.zhangge.ContextListener</listener-class>
</listener>

{% endhighlight %}

主要有这些监听器：

	ServletContextAttributeListener：监听对application属性的操作
	ServletContextListener：监听ServletContext生命周期的变化
	HttpSessionListener：监听HttpSession的操作
	ServletRequestAttributeListener：监听request的参数属性
	ServletRequestListener：监听request

## 6 Filter过滤器

过滤器和struts的拦截器很相似，都是在处理请求之前进行相关的处理，这里是在请求来到servlet之前交给过滤器处理，一般我们做权限控制，地址拦截等等。

实现一个Filter以后，也是在web.xml配置的，所以它也是服务器进行管理生命周期的：

	<filter>
		<filter-name>FilterTimer</filter-name>
		<filter-class>org.zhangge.FirstFilter</filter-class>
	</filter>
	<filter-mapping>
		<filter-name>FilterTimer</filter-name>
		<url-pattern>/*</url-pattern>
	</filter-mapping>

Filter是接口，主要有三个方法:init(), doFilter, destroy()。一般我们实现doFilter即可，代码如下：

{% highlight java %}

public class FirstFilter implements Filter {

    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        System.out.println("init LoginFilter");
    }

    @Override
    public void doFilter(ServletRequest request, ServletResponse response,
            FilterChain chain) throws IOException, ServletException {

        HttpServletRequest req = (HttpServletRequest)request;
        HttpServletResponse resp = (HttpServletResponse)response;

        String comment = req.getParameter("comment");
        comment = comment.replace("Sex", "***");

        req.setAttribute("comment", comment);
        chain.doFilter(request, response);
    }

    @Override
    public void destroy() {
        System.out.println("destroy!!!");
    }
}

{% endhighlight %}

如果需要过滤掉这个请求直接就返回好了，否则就交给过滤链的下一个过滤器进行处理。