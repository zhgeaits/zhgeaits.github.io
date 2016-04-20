---
layout: post
title: "更轻量级的Javaweb应用服务器Jetty"
date: 2011-11-25 01:15:24
categories: javaweb
supertype: career
type: java&web
---

>当时飞哥从淘宝实习回来教会我们使用maven，同时学习使用jetty的，于是所有的项目都使用了jetty。

## 1. 什么是jetty

jetty也是一个javaweb应用服务器，也就是说，还是servlet和jsp的容器。学习过了tomcat以后，不难明白jetty是一个什么东西。当然，这只是功能是的对比，在设计与实现上还是有很大的区别的。为什么我们使用jetty？因为够快，当项目比较大的时候，tomcat启动的时候就很慢了，使用jetty的话，开发起来非常的快。而我们也只是使用到jetty这一点点的功能罢了。项目部署以后还是使用tomcat的。

网上一篇很不错的关于jetty的文章：[传送门](https://www.ibm.com/developerworks/cn/java/j-lo-jetty/ "Jetty 的工作原理以及与 Tomcat 的比较").

jetty的官方网站是：http://www.eclipse.org/jetty。它是开源的！现在是eclipse管的。之前可不是，我们使用的是jetty的6.x版本，看到包名都是org.mortbay开头的，现在是org.eclipse开头了。看到上面的说明jetty的历史，mortbay只是其中的一个Sponsor而已。

>The Java HTTP server that became jetty was originally developed in 1995 by Greg Wilkins of Mort Bay Consulting as part of an issue tracking application. Versions 1.x through to 6.1.x of jetty were developed under org.mortbay packaging and Mort Bay still holds the major part of the copyright on the jetty code base. Mort Bay directly hosted the jetty project until version 3.x and was the prime sponsor of development until 6.x.

关于更详细的版本历史，参考官网。

## 2. 使用jetty

使用很简单，因为我们的项目是用maven构建的web项目，所以，在pom文件配置jetty座位插件使用，如下：

{% highlight xml %}

<plugin>
	<groupId>org.mortbay.jetty</groupId>
	<artifactId>maven-jetty-plugin</artifactId>
	<configuration>
		<contextPath>/${project.artifactId}</contextPath>
		<webDefaultXml>webdefault.xml</webDefaultXml>
		<connectors>
			<connector implementation="org.mortbay.jetty.nio.SelectChannelConnector">
				<port>8081</port>
				<maxIdleTime>60000</maxIdleTime>
			</connector>
		</connectors>
		<requestLog implementation="org.mortbay.jetty.NCSARequestLog">
			<filename>target/request_access.log</filename>
			<retainDays>90</retainDays>
			<append>false</append>
			<extended>false</extended>
			<logTimeZone>GMT+8:00</logTimeZone>
		</requestLog>
		<systemProperties>
			<systemProperty>
				<name>productionMode</name>
				<value>false</value>
			</systemProperty>
		</systemProperties>
	</configuration>
	<version>6.1.26</version>
</plugin>

{% endhighlight %}

最后再命令行运行：`mvn jetty:run -Dmaven.test.skip=true`

关于那个webdefault.xml，忘记是一个什么错误了，如果没有的话会报错的。从以前的项目那里复制过来即可。

以后继续web项目的时候会继续完善记录的~