---
layout: post
title: "MyBatis学习记录"
date: 2012-07-11 01:15:24
categories: javaweb
type: java&web
---

1.以前在spring配置文件配置每个mapper的bean，现在可以用另外一种方式来自动扫描：
{% highlight xml %}
<beanclass="org.mybatis.spring.mapper.MapperScannerConfigurer">
	<propertyname="basePackage"value="org.zhangge.mapper"/>
</bean>
{% endhighlight %}

第一，无需指定引用SqlSessionFactory，因为MapperScannerConfigurer在创建映射器时会通过自动装配的方式来引用。

其次，默认bean名字为首字母小写，也可以在mapper接口使用@Component来修改bean名字。

2.以前需要在mybatis的配置文件sqlmap_configuration.xml配置每一个bean的alias，这样，写sql代码是的resultType就只用一个alias了，如果没有特别的alias的话，直接默认是bean名字的话，那么自动扫描更加方便，使用如下配置即可：
{% highlight xml %}
<!-- 配置mybatis的session -->
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
	<property name="configLocation" value="classpath:sqlmap_configuration.xml" />
	<property name="dataSource" ref="dataSource" />
	<!-- 自动装配bean alias,这样不用再mybatis那里配置 -->
	<property name="typeAliasesPackage" value="org.zhangge.bean" />
</bean>
{% endhighlight %}