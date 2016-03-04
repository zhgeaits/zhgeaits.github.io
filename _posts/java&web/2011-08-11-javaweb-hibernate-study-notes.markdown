---
layout: post
title: "相对较为重量级的ORM框架Hibernate"
date: 2011-08-11 01:15:24
categories: technology
type: java&web
---

>那年暑假，大一，留校学习，Php，javaweb，hadoop，而这是看西安云工厂视频学习的Hibernate。

## 1. 什么是ORM和Hibernate

之前学习JDBC的时候，了解过什么是数据访问层和数据访问对象了，看回之前那边blog笔记，也总结过一个简单的orm框架，而hibernate就是一个orm框架，对erp来讲，它是轻量级的，但是对其我们来讲，对于mybatis这些框架来讲，又较为重量级了！~ORM的全称为Object-Relational Mapping（对象关系映射），即对象和关系之间的映射，我们知道数据库是二维表的关系，而在业务层的数据又是javabean的对象，因此两者之间就需要存在一个映射关系。

把对象存在关系表，或者把关系表记录读取到对象，这些都是繁琐的重复的工作，并且数据源还可以变化的，而hibernate就是这样一个orm框架，帮助我们完成这些操作。本质上还是对jdbc进行了封装。

这也是持久化层，首先，保存在内存的程序数据，程序退出后，数据就消失了，称为瞬时状态；保存在磁盘上的程序数据，程序退出后依然存在，称为程序数据的持久状态；持久化就是将程序数据在瞬时状态和持久状态之间相互转换的机制。

## 2. 如何去学习Hibernate

Hibernate是ssh中的一员，这么的出名，自然网上资料非常的多，而且官方的文档也十分的全面，主要还是英文的，但是也有中文的。关于历史这些东西还是需要网上去科普的。官方网站是：http://hibernate.org/orm/，下载最新的稳定版本（可能会被墙~）。解压以后主要是3个目录，documentation，lib，project；因为这个伟大的项目也是开源的，所以project目录就是源码，lib是我们开发时候需要用到的jar包，而documentation是我们学习的文档，超级全面。这个目录下有，和其他框架整合集成的文档，java类文档，映射指导，用户指导和快速引导。

### 2.1 查看官方的例子

我们去看快速指导目录quickstart，里面有一个hibernate-tutorials.zip压缩包，里面都是一些基础的测试。可惜是没有想struts的文档那样有一个简单的demo来学习的，只能啃那些文档，或者可以去google一下别人的教程。那些文档超级多，而且一般最新版本是没有中文版的。还好，能够简单参考一下hibernate-tutorials.zip里面的basic用法的单元测试，里面是maven构建的，可以暂时忽略。maven依赖了Hibernate需要的jar包，其他的只是多了一个hibernate.cfg.xml配置文件和对象的映射配置文件Event.hbm.xml。

打开hibernate.cfg.xml，如下：

{% highlight xml %}

<hibernate-configuration>

    <session-factory>

        <!-- Database connection settings -->
        <property name="connection.driver_class">org.h2.Driver</property>
        <property name="connection.url">jdbc:h2:mem:db1;DB_CLOSE_DELAY=-1;MVCC=TRUE</property>
        <property name="connection.username">sa</property>
        <property name="connection.password"/>

        <!-- JDBC connection pool (use the built-in) -->
        <property name="connection.pool_size">1</property>

        <!-- SQL dialect -->
        <property name="dialect">org.hibernate.dialect.H2Dialect</property>

        <!-- Disable the second-level cache  -->
        <property name="cache.provider_class">org.hibernate.cache.internal.NoCacheProvider</property>

        <!-- Echo all executed SQL to stdout -->
        <property name="show_sql">true</property>

        <!-- Drop and re-create the database schema on startup -->
        <property name="hbm2ddl.auto">create</property>

        <mapping resource="org/hibernate/tutorial/hbm/Event.hbm.xml"/>

    </session-factory>

</hibernate-configuration>

{% endhighlight %}

可以看到主要是配置了session-factory这个节点，基本的配置包括了数据库和jdbc的连接池数量；然后就是dialect，意思使用myql还是其他的sql语句；还有就是一些开关设置。最后重要的是对象的映射文件配置mapping。

打开Event.hbm.xml，如下：

{% highlight xml %}

<hibernate-mapping package="org.hibernate.tutorial.hbm">

    <class name="Event" table="EVENTS">
        <id name="id" column="EVENT_ID">
            <generator class="increment"/>
        </id>
        <property name="date" type="timestamp" column="EVENT_DATE"/>
        <property name="title"/>
    </class>

</hibernate-mapping>

{% endhighlight %}

可以看到是配置一个class Event，都比较简单，以后这里会需要配置复制的数据类型~

打开Event.java里面就是一个标准javabean而已。

而看基本的一个保存例子，代码如下：

{% highlight java %}

SessionFactory sessionFactory = new MetadataSources( registry ).buildMetadata().buildSessionFactory();

// create a couple of events...
Session session = sessionFactory.openSession();
session.beginTransaction();
session.save( new Event( "Our very first event!", new Date() ) );
session.save( new Event( "A follow up event", new Date() ) );
session.getTransaction().commit();
session.close();

// now lets pull events from the database and list them
session = sessionFactory.openSession();
session.beginTransaction();
List result = session.createQuery( "from Event" ).list();
for ( Event event : (List<Event>) result ) {
	System.out.println( "Event (" + event.getDate() + ") : " + event.getTitle() );
}
session.getTransaction().commit();
session.close();

sessionFactory.close();

{% endhighlight %}

可以学习到一个Session其实就是对数据库的一个连接访问会话，而SessionFactory是用来管理对数据库连接的，里面是可以配置连接池的。我们的操作都是使用session的，首先要打开一个事务，然后操作，最后commit。当然最后不要忘记关闭session了。

### 2.2 模范写一个helloworld例子

1.打开myeclipse，新建一个java project，不需要web project。

2.虽然可以像struts那样使用myeclipse直接支持Hibernate，但是，我们还是自己把jar包复制进去。把下载的Hibernate里面lib下的required和osgi复制到自己的项目。别忘了还有数据库的驱动包，这里使用了mysql。

3.在src目录新建hibernate.cfg.xml，如下：

{% highlight xml %}

<hibernate-configuration>

<session-factory>

	<!-- Database connection settings -->
	<property name="connection.driver_class">
		com.mysql.jdbc.Driver
	</property>
	<property name="connection.url">
		jdbc:mysql://localhost:3306/zhangge_test
	</property>
	<property name="connection.username">root</property>
	<property name="connection.password">mysql_root</property>

	<!-- JDBC connection pool (use the built-in) -->
	<!-- <property name="connection.pool_size">1</property> -->

	<!-- SQL dialect -->
	<property name="dialect">
		org.hibernate.dialect.MySQLDialect
	</property>

	<!-- Enable Hibernate's automatic session context management -->
	<property name="current_session_context_class">thread</property>

	<!-- Disable the second-level cache -->
	<property name="cache.provider_class">
		org.hibernate.cache.NoCacheProvider
	</property>

	<!-- Echo all executed SQL to stdout -->
	<property name="show_sql">true</property>

	<!-- Drop and re-create the database schema on startup -->
	<property name="hbm2ddl.auto">create</property>
	<property name="format_sql">true</property>

	<mapping resource="org/zhangge/Student.hbm.xml" />
	<mapping class="org.zhangge.Teacher" />
</session-factory>

</hibernate-configuration>

{% endhighlight %}

4.Student是一个标准的javabean，而Teacher则是使用了注解的方式，Student.hbm.xml如下：

{% highlight xml %}

<hibernate-mapping package="org.zhangge">

	<class name="Student" table="student" >
		<id name="id" column="id">
			<generator class="native"/>
		</id>
        <property name="name" column="name"></property>
        <property name="age" column="age"></property>

    </class>
</hibernate-mapping>

{% endhighlight %}

Teacher.java如下：

{% highlight java %}

@Entity
public class Teacher {
	private int id;
	private String name;
	private String title;

	public Teacher() {
		super();
	}

	@Id
	@GeneratedValue
	public int getId() {
		return id;
	}
	public void setId(int id) {
		this.id = id;
	}
	public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}
	public String getTitle() {
		return title;
	}
	public void setTitle(String title) {
		this.title = title;
	}
}

{% endhighlight %}

5.编写一个Test类使用：

{% highlight java %}

public class TestHibernate {
	private SessionFactory sessionFactory;
	
	public static void main(String[] args) {
		TestHibernate test = new TestHibernate();
		test.sessionFactory = new Configuration().configure().buildSessionFactory();
		
		test.testStudent();
		test.close();
	}
	
	private void close() {
		sessionFactory.close();
	}
	
	private void testStudent() {
		Session session = sessionFactory.getCurrentSession();
		session.beginTransaction();
		Student student1 = new Student();
		student1.setName("zhangge");
		session.save(student1);
		session.getTransaction().commit();
	}
	
	public void testTeacherSave() {
		Teacher teacher = new Teacher();
		teacher.setTitle("paoKing9");
		teacher.setId(17);
		teacher.setName("paoGe9");

		Session session = sessionFactory.getCurrentSession();
		session.beginTransaction();
		session.save(teacher);
		session.getTransaction().commit();
		if (session != null && session.isOpen()) {
			session.close();
		}
	}

	public void testTeacherGet() {
		Session session = sessionFactory.getCurrentSession();
		session.beginTransaction();
		Teacher teacher = (Teacher) session.get(Teacher.class, 17);
		System.out.println(teacher.getName());
		session.getTransaction().commit();
	}

	public void testTeacherLoad() {
		Session session = sessionFactory.getCurrentSession();
		session.beginTransaction();
		Teacher teacher = (Teacher) session.load(Teacher.class, 17);
		System.out.println(teacher.getName());
		session.getTransaction().commit();
	}

	public void testTeacherUpdate() {
		Session session = sessionFactory.getCurrentSession();
		session.beginTransaction();
		Teacher teacher = (Teacher) session.load(Teacher.class, 17);
		teacher.setName("zhanglaoshi");
		session.getTransaction().commit();
	}
}

{% endhighlight %}

## 3. 更多

这只是最简单的使用，Hibernate是很复杂的东西，里面很多复杂的设计思想，关于数据的即使加载，延迟加载，对象的状态，HQL，查询条件，排序，对象的关系，不要出现笛卡尔积，更多的注解使用，更多的api使用等等，如何和spring，struts整合等等。

可以去看做过的web外包项目，学校的项目。如果以后工作会跟这个相关，那么再去学习吧~再来这里记录吧~