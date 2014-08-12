---
layout: post
title: "Tomcat无法启动问题"
date: 2013-07-02 16:18:30
categories: tomcat
type: other
---

>材料库部署以后，本来是在ubuntu server + tomcat6 + jdk6的环境，但是，客户的各种原因导致现在是放在了win2003server下，而且又不能登录来调试，出问题比较麻烦。
>
>今天又出问题了，tomcat无法启动，于是叫客户发了tomcat的日志过来，发现出现这个问题：

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

>再次询问才知道原来是把jdk升级到7了，jvm.dll已经改变路径了，然后只能目前叫客户在tomcat的配置那里修改重新指向了，其实也可以重新安装jre6的。
>
>另外[Google]了其他的解决办法。

[Google]: http://www.mkyong.com/tomcat/tomcat-error-prunsrvc-failed-creating-java-jvmdll
