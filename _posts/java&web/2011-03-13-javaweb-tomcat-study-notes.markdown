---
layout: post
title: "轻量级的Javaweb应用服务器Tomcat学习记录"
date: 2011-03-13 11:15:24
categories: javaweb
type: java&web
---

## 1 什么是应用服务器Tomcat

当前的互联网应用架构基本上都是Brower/Server架构的，即web应用都是基于http协议开发的，这里要注意区分网络编程的概念，网络编程说的是基于TCP/IP协议的开发。而server就是web应用服务器了，也就是说它的主要功能是用来响应web的http请求的。因此我们用java编写的web应用就需要运行在一个server里面了；实际上，java编写的web程序基本上都是servlet和jsp，而tomcat就是运行编译后的servlet的class容器。当一个请求来了的时候，tomcat会处理这个请求url，最后分配到响应的servlet或者jsp来执行响应的逻辑，然后返回结果响应页面。

Tomcat是Apache软件基金会的，所以它是开源免费的，官网是：http://tomcat.apache.org/。百科上说Tomcat是apache服务器的扩展，并且说Tomcat处理静态HTML的能力不如Apache服务器，当配置正确时，Apache为HTML页面服务，而Tomcat实际上运行JSP页面和Servlet；这点还是不太明白，因为也没有认真研究过，不过，以前工作室的网络架构还算比较正确，一台服务跑apache，然后逆向代理到tomcat服务器，也就是说一般的网页处理是在apache的，而java的web是在另一台服务器的tomcat的。