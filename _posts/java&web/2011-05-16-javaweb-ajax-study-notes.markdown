---
layout: post
title: "异步刷新技术"
date: 2011-05-16 01:15:24
categories: javaweb
type: java&web
---

## 1. 什么是Ajax

学习web的时候，ajax是一个重点，因为，对于刚入大学的我们，缺乏太多的基础了，不要说网络方面的，就连同步和异步的理解都会费一番功夫。就如标题所云，这是一门异步刷新技术，相比同步，异步就是不会阻塞，在后台的线程去运行，请求网络，进而得到网络响应以后就可以局部刷新界面了。一般来说，我们在网页commit一个action以后，会发送请求到服务器，然后整个页面都进入阻塞状态，等待刷新，一旦有回来的响应以后，也就是整个页面回来了，即全面刷新。同步固然易于理解，不过不能适合复杂的功能，我们大部分情况都需要使用异步来开发的。试想一下，用户在观看这个页面的部分内容的时候，同时在异步刷新下面的内容，而不会影响到当前的阅读，那是多好的体验啊。

AJAX的全称是：Asynchronous Javascript And XML；异步的JavaScript和XML，是指一种创建交互式网页应用的网页开发技术。这并不是什么难的东西，本质就是浏览器提供了一些异步的组件（如xmlHttpRequest）给你使用，而这些组件的本质就是在后台开辟了新的线程进行网络请求，最后通过回调函数与你交互（关于回调，也是一个有意思的东西，在js里面，回调的单位是函数，而java则是对象）。当我们的编程经验越来越丰富的时候，便会明白这一切了。不管是web开发得ajax，还是其他平台的异步组件，如windows，android，ios等等，本质都是线程的。因此，最重要的还是计算机操作系统的基础要牢牢掌握。

还记得那时候我称为“阿贾克斯”技术，哈哈，想想，也会觉得我们那时候对技术的一种痴迷，但愿永远保持那颗炽热的心，不要让热情磨灭。不断进步，不断理解技术背后的本质与人生。通常，我们都不会直接使用原生的js的ajax组件了，因为需要适配不同的浏览器，这些重复的工作，当然是交给框架来做，最出色的当然是jquery了。具体的学习笔记，参考js那篇blog。

### 1.1 缓存问题

有时候因为缓存的问题，导致一直请求都是返回同样的结果，本质是浏览器的缓存问题，解决方法当然可以禁用浏览器的缓存，或者使用post请求，但是，这绝对不是好办法，也会导致网页速度变慢；另外，在服务端是可以在http的响应头加上一些设置参数说明不使用缓存的，如：

>response.addHeader("pragma", "no-cache");  
>response.addHeader("cache-control", "no-cache");  
>response.addHeader("expires", "0");

这个就是遵循了http的规范的。还有一个方法，浏览器缓存的key一般是url，只要我们每次url都不一样就可以，使用不同的url可以在url最后每次都加上一个时间戳字段。但是会不会缓存越来越大？

## 2. 通用的JSON格式数据

JSON数据格式，现在感觉就像成为了一个行业内的默认规范了，哪里都会使用到，这样的设计，真是永不过时，可谓之优美也！

json的全称是：JavaScript Object Notation。可以看出是js的一个对象标记，本质就是一组字符串而已，用花括号包围起来，即使一个对象，里面存储的数据是以key-value形式存在。而用中括号包围的则是数组。key一般是字符串，而value则可以是字符串，对象和数组；数组又可以是字符串，对象，或者数组构成。例子：

>{"keyhere" : "valuehere"}  
>{"superkey" : {"keyhere" : "valuehere"}}  
>{"arraykey" : ["zhangge", "handsome boy"]}  
>{"arraykey" : [{"keyhere1" : "valuehere1"}, {"keyhere2" : "valuehere2"}]}

极之简单，仅此而已！

## 3. 优秀的异步网络框架DWR

开发web的时候用到json，纯粹的servlet，jsp不能把对象转换成json对象，必须我们自己实现，又是框架性的工作了，当然是交个框架来做了。

### 3.1 什么是DWR

DWR的全称是：Direct Web Remoting，这个是它的官网：http://directwebremoting.org/dwr/index.html。官网上面包含了详细的文档和demo，完全可以从那里自行学习，虽然刚开始的时候看英文比较慢，但是逐渐地的会越来越快，并且dwr还是开源的。下载dwrdemo.war就可以看到里面所有的使用例子了。

### 3.2 配置使用

1.打开myeclipse新建一个web项目

2.把下载的dwr.jar添加进去，同时还是需要commons-logging.jar的；当然完全可以使用maven进行依赖的。

3.配置web.xml，只不过是添加一个servlet而已，如下：

{% highlight xml %}

<servlet>
  <display-name>DWR Servlet</display-name>
  <servlet-name>dwr-invoker</servlet-name>  
  <servlet-class>org.directwebremoting.servlet.DwrServlet</servlet-class>
  <init-param>
     <param-name>debug</param-name>
     <param-value>true</param-value>
  </init-param>
</servlet>

<servlet-mapping>
  <servlet-name>dwr-invoker</servlet-name>
  <url-pattern>/dwr/*</url-pattern>
</servlet-mapping>

{% endhighlight %}

其实就可以大概猜到，肯定是通过拦截url，然后有一个处理的servlet，简单的原理流程就是这样。

4.在WEB-INF下新建一个dwr.xml的配置文件，如下：

{% highlight xml %}

<!DOCTYPE dwr PUBLIC
    "-//GetAhead Limited//DTD Direct Web Remoting 3.0//EN"
    "http://getahead.org/dwr/dwr30.dtd">

<dwr>
  <allow>
    <create creator="new" javascript="JDate">
      <param name="class" value="java.util.Date"/>
    </create>
    <create creator="new" javascript="Demo">
      <param name="class" value="org.zhangge.MyDWRBean"/>
    </create>
  </allow>
</dwr>

{% endhighlight %}

更详细的配置使用，可以参考文档，特别是返回的数据类型怎么转换，怎么使用json。

5.创建一个javabean：

{% highlight java %}

public class MyDWRBean {

	public String sendMessage(int id, String callback) {
		return id + callback;
	}
}

{% endhighlight %}

6.启动tomcat，然后在浏览器运行：http://localhost:8080/TestDWR/dwr/。这时候见到dwr的页面就成功了，里面会有详细的教程怎么使用的。

7.在jsp页面添加：

{% highlight javascript %}

<script type='text/javascript' src='/TestDWR/dwr/engine.js'></script>
<script type='text/javascript' src='/TestDWR/dwr/interface/Demo.js'></script>
<script type='text/javascript' src='/TestDWR/dwr/util.js'></script>

{% endhighlight %}

8.然后这样调用：

{% highlight javascript %}

var callbackfun = function(data)
{
  alert(data);
}

Demo.sendMessage(27, "zhangge is handsome"), callbackfun);

{% endhighlight %}

### 3.3 论DWR和RPC

它可以允许在浏览器里的代码使用运行在WEB服务器上的JAVA函数，就像它就在浏览器里一样。多么的简单，它是封装了ajax的技术，进行rpc的使用，去调用普通的java类。rpc就是一个远程方法的调用，可以认为使用的ajax的请求，并且带有json数据传输的，都是web类型的rpc，一般开发，定义好接口，即url，客户端就对接口进行开发，其实也是远程的方法调用了，客户端不用关心后端怎么处理的。通常来说，使用jquery和springmvc，struts这些就足够了，它们已经可以配置返回的数据就是json格式的。但是如果结合dwr，会让我们的开发更加的方便，即直接调用javabean，而不是一个action。具体dwr怎整合到这些框架，和怎么使用是最方便的，以后用得多了，慢慢来记录吧！~