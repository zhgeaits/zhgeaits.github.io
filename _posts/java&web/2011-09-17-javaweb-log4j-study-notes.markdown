---
layout: post
title: "Java中最流行的日志框架log4j学习记录"
date: 2011-09-17 01:15:24
categories: javaweb
type: java&web
---

## 1. 什么是Log4J

当我们开发项目的时候，log日志是少不了的，它会帮助我们去了解程序的运行情况和排查问题。一般我们的日志需要写到文件，所以java里面的纯粹写到控制台的类是没法使用的。那么java就需要一个log的组件，log4j便出来了，它的意思是log for java。它是apache的开源项目，官方网站是：http://logging.apache.org/log4j，需要学习当然是从官网上来了。有充分的文档和例子。

## 2. log4j和commons-logging的关系

通常我们使用log4j肯定也会使用到commons-logging的jar包，同时两个都依赖，为什么呢？commons-logging也是apache的，看名字就知道是通用的logging，其实是一套接口规范，而log4j只是其中的一个实现，因此，还是可以换其他的log框架的。也就是说，我们编写代码的时候是依赖commons-logging的接口编写的，当运行起来的时候就会去classpath中寻找相关的实现类，如果没有的话就会报错，加入log4j或者其他的日志框架都可以的。这就是面向接口编程的最好例子呢！

## 3. log4j的组件

主要有三个组件，分别是 Logger，Appender和Layout，理解了，使用起来也不困难。Logger就是日志器，我们只要调用这个类的接口进行使用。appender是用来指定日志输出到哪里的，可以是控制台，文件等。layout就是格式化日志信息了，可以是文本文件的格式化，也可以是html网页文件的格式化。

### 3.1 日志等级

日志等级有5个级别，分别是DEBUG, INFO, WARN, ERROR 和 FATAL。即，调试，信息，警告，错误，致命错误。这些的级别大小排列如下：DEBUG < INFO < WARN < ERROR < FATAL。我们每打一条日志都需要设置这条日志的级别是什么。而我们配置appender的时候，需要配置输出的级别是什么，即只有是当前级别，或比当前级别还要大的日志才能输出。例如，我配置的appender.level=info。那么log.debug()的日志就不会被输出，其他的都会被输出了。

## 4. 配置使用

日志的使用都比较简单，基本上是拿到一个对象引用就可以了，如：

>Logger logger = LogFactory.getLog(ZhanggeBean.class);

复杂的地方在于配置，和整合到其他的框架中去，具体都是可以去看官方的文档，或者参考我以前做的web项目，如stm，clk，cmms等，所有的项目都是用到了log4j的。这里就摆出一个配置例子，以后继续web开发的时候会去深入学习的。

### 4.1 配置

它的配置可以是properties文件，也可以是xml，我们一般使用properties的。

{% highlight xml %}

log4j.rootLogger=ERROR,stdout,fileout

log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=[%d{yyyy-MM-dd HH:mm:ss:SSS}][%C:%M] %m%n

log4j.appender.fileout=org.apache.log4j.DailyRollingFileAppender
log4j.appender.fileout.File=${webapp.root.clk}/logs/clk 
log4j.appender.fileout.DatePattern=yyyy-MM-dd'.html'
log4j.appender.fileout.layout=org.apache.log4j.HTMLLayout

log4j.logger.org.xhome.clk=INFO
log4j.logger.org.xhome.clk.intercept=ERROR

{% endhighlight %}

更多详细的使用，去官网看，当然可以直接google，也可以看以前做的项目。

### 4.2 使用

当然log4j不仅仅是用在web项目中，只要是java项目都可以使用。

一般情况下的使用，都要调用一下log.isDebugEnabled()来判断等级，这样可以提升不少的性能，减少拼字符串的操作等等。