---
layout: post
title: "伟大的框架SpringFramework"
date: 2011-08-15 01:15:24
categories: javaweb
supertype: career
type: java&web
---

## 1 什么是SpringFramework

做web开发得ssh框架中最厉害的一个就是spring了，里面的设计思想很多。它也还是一个相对轻量级的框架，因为相对ejb来讲，还是很小型的。spring很多东西要学习，我就只学了其中的几个而已，jdbctemplate，mvc，ioc，aop。它能干什么？起初它只是一个容器，用来管理javabean的，后来，它发展成为了很大的组织，开发了很多的模块，有很多功能，简直可以取缔struts和Hibernate了，作为一个完整的ejb。

## 2 配置使用

官方网站是：http://projects.spring.io/spring-framework/。它不像Hibernate和struts那样直接下载一个完整的包，它是开源的，交个github来托管，它没有提供构建包下载了，需要用maven或者gradle或者ant来依赖使用，或者自行下载源码进行构建。但是官方是直接有文档的。

不过还是可以从这里进行下载的：http://repo.springsource.org/libs-release-local/org/springframework/spring/。

### 2.1 基本的IOC功能

它不会有demo，只能去看文档，不过使用起来比较简单。

1.在myeclipse上新建一个java project，虽然myeclipse可以直接支持spring，但是我们还是自己复制下载的jar进去。

2.但是spring需要log系统，所以需要去apache那里下载http://commons.apache.org/proper/commons-logging。

3.编写两个标准的javabean，如下：

{% highlight java %}

public class Btd {
	private int y;
	private int m;
	private int d;
	public int getY() {
		return y;
	}
	public void setY(int y) {
		this.y = y;
	}
	public int getM() {
		return m;
	}
	public void setM(int m) {
		this.m = m;
	}
	public int getD() {
		return d;
	}
	public void setD(int d) {
		this.d = d;
	}
	public Btd(int y, int m, int d) {
		super();
		this.y = y;
		this.m = m;
		this.d = d;
	}
	@Override
	public String toString() {
		return "Btd [y=" + y + ", m=" + m + ", d=" + d + "]";
	}
	public Btd() {
		super();
	}
}

public class Student {
	private String name;
	private int age;
	private Btd btd;
	
	@Override
	public String toString() {
		return "Student [name=" + name + ", age=" + age + ", btd=" + btd + "]";
	}
	public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}
	public int getAge() {
		return age;
	}
	public void setAge(int age) {
		this.age = age;
	}
	public Btd getBtd() {
		return btd;
	}
	public void setBtd(Btd btd) {
		this.btd = btd;
	}
	public Student() {
		super();
	}
}

{% endhighlight %}

4.在src目录新建applicationContext.xml，内容如下：

{% highlight xml %}

<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:p="http://www.springframework.org/schema/p"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">

	<bean id="btd" class="org.zhangge.Btd">
		<property name="y" value="2011"></property>
		<property name="m" value="5"></property>
		<property name="d" value="5"></property>
	</bean>
	<bean id="student" class="org.zhangge.Student">
		<property name="age" value="23"></property>
		<property name="name" value="zhangsi"></property>
		<property name="btd" ref="btd"></property>
	</bean>
</beans>

{% endhighlight %}

5.编写一个测试代码，如下：

{% highlight java %}

ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
Btd btd = (Btd) context.getBean("btd");
System.out.println(btd);
Student stu = context.getBean("student", Student.class);
System.out.println(stu);

{% endhighlight %}

由上面可以我们不需要再自己管理javabean了，都是交给spring来创建并且set到我们需要的地方去。这就是他的容器功能和ioc啊，依赖别人来注入，控制反转了嘛~感觉好像没什么必要似的，但是一但项目大起来的时候，你就会发现重要了。我只有一个抽象的属性，不需要知道注入的是什么实现类对象。

更多详细注入类型，以后用到再来这里记录补充。

## 3 高级功能AOP

什么是aop，它是Aspect-Oriented Programming，面向切面的编程。意思是垂直我们平时正常的业务流程来写代码，是不需要改变原来的代码的。例如，打印日志，安全验证，事务处理等等，这些都不是很必要的逻辑，却与业务高度耦合在一起了，不利于程序开发和维护。这就需要用到java的动态代理设计模式了，需要去先学习一下java的动态代理。完整的项目可以去看我之前在沈阳做的那个项目。

然后去看以前在学校做的web外包项目，就知道spring怎么跟struts，Hibernate，mybatis整合的，基本上就能学会spring的基本功能了，主要知道一下注解，xml的方式，aop的配置，事务管理aop等等。以后如果工作还是web的话，再来完善。

## 4 web模块SpringMVC

这是spring的mvc模块，与struts2的最大区别在于，它的一个action是一个controller，一个方法才是一个请求；而struts2的一个action是一个请求。详细的使用看以前web的外包项目cmms，托管在bitbucket上面。因为比较简单，就不再记录了，以后如果工作还是web的话，再来完善。

## 5 ORM模块JDBCTemplate

这是就是封装了jdbc的一些操作，之前的jdbc那个blog就是模范这个来写的，沈阳那个项目，那些练习servlet和jsp的项目也是用到了这个的，都可以去回顾温习。还有参加花旗杯的项目也是用到了这个的。

spring+mybatis是cmms项目；struts2+spring+Hibernate是cabonsystem项目；struts2+spring+mybatis是clk项目；都在bitbucket上面托管，皆可以去温习回顾！~
