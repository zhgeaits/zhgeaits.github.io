---
layout: post
title: "轻量级的Javaweb应用服务器Tomcat"
date: 2011-03-13 11:15:24
categories: javaweb
type: java&web
---

## 1 什么是应用服务器Tomcat

当前的互联网应用架构基本上都是Brower/Server架构的，即web应用都是基于http协议开发的，这里要注意区分网络编程的概念，网络编程说的是基于TCP/IP协议的开发。而server就是web应用服务器了，也就是说它的主要功能是用来响应web的http请求的。因此我们用java编写的web应用就需要运行在一个server里面了；实际上，java编写的web程序基本上都是servlet和jsp，而tomcat就是运行编译后的servlet的class容器。当一个请求来了的时候，tomcat会处理这个请求url，最后分配到响应的servlet或者jsp来执行响应的逻辑，然后返回结果响应页面。

Tomcat是Apache软件基金会的，所以它是开源免费的，官网是：http://tomcat.apache.org/。百科上说Tomcat是apache服务器的扩展，并且说Tomcat处理静态HTML的能力不如Apache服务器，当配置正确时，Apache为HTML页面服务，而Tomcat实际上运行JSP页面和Servlet；这点还是不太明白，因为也没有认真研究过，不过，以前工作室的网络架构还算比较正确，一台服务跑apache，然后逆向代理到tomcat服务器，也就是说一般的网页处理是在apache的，而java的web是在另一台服务器的tomcat的。

网上一篇很不错的关于tomcat的文章：[传送门](http://www.ibm.com/developerworks/cn/java/j-lo-tomcat1/ "Tomcat 系统架构与设计模式").

## 2 安装配置

当时tomcat出到7.0版本的时候我就已经没再跟进了。到官网下载一个7.0版本，解压，配置环境变量就完成了，windows和linux，mac都一样的道理。就算是新的版本8.0也不会差太多，只要看RUNNING.txt这个文档即可。

windows下载以后解压，然后配置环境变量如下：

>CATALINA_HOME=D:\develop\apache-tomcat-6.0.32

然后运行这个bat即可启动：

>$CATALINA_HOME\bin\startup.bat

在浏览器输入地址`127.0.0.1:8080`后成功登陆tomcat服务器；但是无法进入manager页面，需要配置conf/tomcat-users.xml这个文件，打开以后发现都是注释掉了的，注意到两个note如下：

>NOTE:  By default, no user is included in the "manager-gui" role required
  to operate the "/manager/html" web application.  If you wish to use this app,
  you must define such a user - the username and password are arbitrary.

>NOTE:  The sample user and role entries below are wrapped in a comment
  and thus are ignored when reading this file. Do not forget to remove
  <!.. ..> that surrounds them.

意思是说想要进入/manager/html管理web应用的用户必须是manager-gui这个角色的，但是默认没有用户是这个角色的，所以如果要使用的话就必须定义这样的角色和用户。另外下面注释掉的是示例，应该移除注释进行定义。

所以，我们打开注释，重新定义如下：

{% highlight xml %}

<role rolename="tomcat"/>
<role rolename="role1"/>
<role rolename="manager-gui"/>
<user username="tomcat" password="tomcat" roles="tomcat"/>
<user username="both" password="tomcat" roles="tomcat,role1"/>
<user username="role1" password="tomcat" roles="role1"/>
<user username="admin" password="admin" roles="manager-gui,tomcat,role1"/>

{% endhighlight %}

最后退出tomcat并重新启动，输入admin admin就可以进入manager页面了。

## 3 部署项目到tomcat

### 3.1 运行中部署war包

如果项目已经打包成war包，那么进入tomcat的manager页面以后可以选择war包进行上传部署，也可以设置一个服务器的目录进行部署。

### 3.2 开发中部署

在MyEclipse中开发的时候，当配置好了tomcat，然后一件deploy就可以把项目部署到了tomcat，然后启动tomcat即可。实际上，是把项目的WebRoot目录copy到了webapps目录而已。而work目录实际是编译webapps目录的结果。

另外也可以配置tomcat指向你的项目目录，这样就不用每次都copy了。在conf/servers.xml的Host节点下添加如下：

	<Context path="/Test" docBase="C:\Users\Administrator\Workspaces\MyEclipse 2015 CI\Test\WebRoot" debug="0" privileged="true">
	</Context>

还可以在conf\Catalina\localhost目录下新建一个xml文件，名字随便取，然后添加上面的内容即可。详细都可以直接上官网看tomcat的文档。

## 4 其他配置

### 4.1 配置端口

默认端口是8080，其实也可以修改的，在conf/servers.xml文件里面有这行配置：

	<Connector port="8080" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443" />

修改即可。

## 5 TroubleShooting

### 5.1 无法启动问题

材料库部署以后，本来是在ubuntu server + tomcat6 + jdk6的环境，但是，客户的各种原因导致现在是放在了win2003server下，而且又不能登录来调试，出问题比较麻烦。今天又出问题了，tomcat无法启动，于是叫客户发了tomcat的日志过来，发现出现这个问题：

{% highlight bash %}
[2013-07-02 13:44:15] [info]  [ 2300] Commons Daemon procrun (1.0.15.0 32-bit) started
[2013-07-02 13:44:15] [info]  [ 2300] Running 'Tomcat6' Service...
[2013-07-02 13:44:15] [info]  [ 6056] Starting service...
[2013-07-02 13:44:15] [error] [ 6056] Failed creating java C:\Program Files\Java\jre6\bin\client\jvm.dll
[2013-07-02 13:44:15] [error] [ 6056] 系统找不到指定的路径。
[2013-07-02 13:44:15] [error] [ 6056] ServiceStart returned 1
[2013-07-02 13:44:15] [error] [ 6056] 系统找不到指定的路径。
[2013-07-02 13:44:15] [info]  [ 2300] Run service finished.
[2013-07-02 13:44:15] [info]  [ 2300] Commons Daemon procrun finished
{% endhighlight %}

再次询问才知道原来是把jdk升级到7了，jvm.dll已经改变路径了，然后只能目前叫客户在tomcat的配置那里修改重新指向了，其实也可以重新安装jre6的。

另外[Google]了其他的解决办法。

[Google]: http://www.mkyong.com/tomcat/tomcat-error-prunsrvc-failed-creating-java-jvmdll