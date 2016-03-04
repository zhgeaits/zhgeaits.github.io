---
layout: post
title: "轻量级的ORM框架MyBatis"
date: 2012-07-11 01:15:24
categories: javaweb
type: java&web
---

## 1. 什么是MyBatis

这是一个持久化层的ORM框架，它主要完成的功能和Hibernate是一致的，但是实现完全不同，Mybatis更加轻量级一些。关于什么是ORM可以去看jdbc和Hibernate的那两篇blog。

它的前身是ibatis，是开源的，后来迁移到googlecode上面以后改名为mybatis，现在代码又迁移到了github上面。官方网站是：http://blog.mybatis.org/。

总体上来说，它和Hibernate的最大区别是，对数据库的操控sql语句交回给了开发者。表与表之间的映射关系等等，都是通过xml文件来配置，用户自己来写sql语句，自己设置查询回来的结果集等等。不像Hibernate那样，基本上的配置只有javabean，然后其他事情，例如编写sql语句等等都是封装好了的（当然，也是可以自己写sql的，但是这并不是它的特点），都是通过一个session来操作。而且javabean不会耦合在一起，mybatis是通过xml来自行配置，指定对应的属性的。更重要的是，整个dao层都可以不用实现了，只需要定义mapper的接口即可，其他的事情都是mybatis来生成代理对象的。

## 2. 如何去学习

通过官方网站，可以进行自行的学习，上面的文档非常的全面，而且还有中文版的。现在最新的还是mybatis3，说明已经很稳定了！我们就下载即可，当然也是可以通过maven来依赖的。我之前的所有外包基本都是mybatis实现的，stm，clk，cmms等等，可以回去温习！

## 3. 且看一个例子

看一个入门例子，基本上什么都会了，然后不懂就去看文档！下载好了以后把相应的jar包拷过去项目中即可。

### 3.1 首先创建一下数据库表

{% highlight mysql %}

create table Student (
	id int primary key auto_increment,
	username varchar(32),
	password varchar(32)
);

{% endhighlight %}

然后在classpath下新建一个mysql.properties

{% highlight mysql %}

driver=com.mysql.jdbc.Driver
url=jdbc\:mysql\://127.0.0.1\:3306/mybatis_practice?characterEncoding\=UTF-8&amp;zeroDateTimeBehavior\=converToNull
username=root
password=123456

{% endhighlight %}

### 3.2 创建mybatis的配置文件

在classpath下创建Configuration.xml，其实名字是随便的，内容如下：

{% highlight xml %}

<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN" 
"http://mybatis.org/dtd/mybatis-3-config.dtd ">

<!-- 
注意：每个标签必须按顺序写，不然蛋疼的DTD会提示错误：The content of element type "configuration" must match 
"(properties?,settings?,typeAliases?,typeHandlers?,objectFactory?,objectWrapperFactory?,plugins?,environments?,mappers?)".
 -->
<configuration>
	<!-- 属性配置 -->
	<properties resource="org/zhangge/mybatis/mysql.properties">
		<!-- 相同属性:最高优先级的属性是那些作为方法参数的，然后是资源/url 属性，最后是 properties元素中指定的属性 -->
		<property name="username" value="root"/>
		<property name="password" value="123456"/>
	</properties>
	
	<!-- 设置缓存和延迟加载等等重要的运行时的行为方式 -->
	<settings>
		<!-- 设置超时时间，它决定驱动等待一个数据库响应的时间  -->
		<setting name="defaultStatementTimeout" value="25000"/>
	</settings>
	
	<!-- 别名 -->
	<typeAliases>
		<typeAlias alias="Student" type="org.zhangge.mybatis.bean.Student"/>
	</typeAliases>
	
	<environments default="development">
		<!-- environment 元素体中包含对事务管理和连接池的环境配置 -->
		<environment id="development">
			<transactionManager type="JDBC" />
			
			<!-- 
				type分三种：
					UNPOOLED是每次被请求时简单打开和关闭连接 
					UNPOOLED的数据源仅仅用来配置以下 4 种属性driver，url，username，password
					POOLED ：JDBC连接对象的数据源连接池的实现，不直接支持第三方数据库连接池
			-->
			<dataSource type="POOLED">
				<property name="driver" value="${driver}" />
				<property name="url" value="${url}" />
				<property name="username" value="${username}" />
				<property name="password" value="${password}" />
			</dataSource>
		</environment>
	</environments>
	
	<!-- ORM映射文件 -->
	<mappers>
		<mapper resource="org/zhangge/mybatis/bean/StudentMap.xml"/>
	</mappers>
</configuration>

{% endhighlight %}

可以看到主要是数据源，mapper，javabean的配置。数据库主要是读取mysql.properties，然后是一些连接池，超时等设置。至于javabean就是标准的javabean了，对应数据库表即可。mappers的配置下面讲起。

### 3.3 新建一个Mapper接口

{% highlight java %}

public interface StudentMapper {

	public Integer insertStudent(Student student);
	
	public Integer updateStudent(Student student);
	
	public List<Student> selectStudents();
	
	public Student selectStudentById(int id);
}

{% endhighlight %}

我并不需要实现这些接口，只需要去适配实现这些接口的sql语句，传递参数的转换和返回参数的转换即可，实际上类的实现，数据库的连接等管理问题交给了mybatis。

于是，在classpath下新建这样一个StudentMap.xml文件：

{% highlight xml %}

<!DOCTYPE mapper     
PUBLIC "-//ibatis.apache.org//DTD Mapper 3.0//EN"     
"http://ibatis.apache.org/dtd/ibatis-3-mapper.dtd">  

<!-- namespace用于java代码调用时识别指定xml的mapper文件 -->
<mapper namespace="org.zhangge.mybatis.mapper.StudentMapper">
	<!-- 配置ORM映射 -->
	<resultMap type="Student" id="student">
		<id property="id" column="id"/>
		<result property="username" column="username"/>
		<result property="password" column="password"/>
	</resultMap>
	
	<!-- 用来定义可重用的SQL代码段 -->
	<sql id="demo_sql">
		username, password
	</sql>
	
	<insert id="insertStudent" parameterType="Student">
		insert into Student(<include refid="demo_sql"/>) values(#{username}, #{password})
	</insert>
	
	<update id="updateStudent" parameterType="Student">
		update Student set username=#{username}, password=#{password} where id=#{id}
	</update>
	
	<select id="selectStudents" useCache="false" flushCache="true" resultMap="student">
		select * from Student
	</select>
	
	<select id="selectStudentById" parameterType="int" resultType="student">
		select * from Student where id=#{id}
	</select>
</mapper>

{% endhighlight %}

这里就用上了Configuration.xml的Alias配置。可以观察到，主要有insert，update和delete等标签，他们的id就是对应到了mapper的接口名字。而这些标签重要的是配置parameterType和resultMap，分别是sql语句的参数和返回结果集。上面的resultmap标签就是配置了一个自定义的返回结果集。当sql语句比较多，复杂，重复的时候还可以用sql标签来定义。

### 3.4 使用入口

有了上面那些代码，那么怎么启动mybatis来使用呢？新建一个SessionFactoryUtil如下：

{% highlight java %}

public class SessionFactoryUtil {
	private static final String RESOURCE="org/zhangge/mybatis/Configuration.xml";
	private static SqlSessionFactory sqlSessionFactory = null;
	private static ThreadLocal<SqlSession> threadLocal = new ThreadLocal<SqlSession>();
	static {
		Reader reader = null;
		try {
			reader = Resources.getResourceAsReader(RESOURCE);
		} catch (IOException e) {
			throw new RuntimeException("Get resource error:" + RESOURCE, e);
		}
		sqlSessionFactory = new SqlSessionFactoryBuilder().build(reader);
	}
	
	/**
	 * Function  : 获得SqlSessionFactory
	 * @author   : bless<505629625@qq.com>
	 * @return
	 */
	public static SqlSessionFactory getSqlSessionFactory(){   
        return sqlSessionFactory;   
    }
	
	/**
	 * Function  : 重新创建SqlSessionFactory
	 * @author   : bless<505629625@qq.com>
	 */
	public static void rebuildSqlSessionFactory(){
		Reader reader = null;
		try {
			reader = Resources.getResourceAsReader(RESOURCE);
		} catch (IOException e) {
			throw new RuntimeException("Get resource error:"+RESOURCE, e);
		}
		sqlSessionFactory = new SqlSessionFactoryBuilder().build(reader);
	}
	
	/**
	 * 
	 * Function  : 获取sqlSession
	 * @author   : bless<505629625@qq.com>
	 * @return   : SqlSession
	 */
	public static SqlSession getSession(){
		SqlSession session = threadLocal.get();
		
		if(session==null){
			if(sqlSessionFactory == null){
				rebuildSqlSessionFactory();
			}
			//如果sqlSessionFactory不为空则获取sqlSession，否则返回null
			session = (sqlSessionFactory!=null) ? sqlSessionFactory.openSession(): null;
		}
		threadLocal.set(session);
		return session;
	}
	
	/**
	 * Function  : 关闭sqlSession
	 * @author   : bless<505629625@qq.com>
	 */
	public static void closeSession(){
		SqlSession session = threadLocal.get();
		session.commit();
		threadLocal.set(null);
		if(session!=null){
			session.close();
		}
	}
}

{% endhighlight %}

这里先不要管ThreadLocal，这个不是重点，它是用来保证线程安全的，假设有多个线程使用SessionFactoryUtil这个对象，那么多个线程拿到的session都是不同的，即不同的线程都是单独打开了不同的数据库会话。

这我们看到其实，mybatis也是通过session来对数据进行访问的。OK，写一个测试类：

{% highlight java %}

public class StudentService {
	public void insertStudent() {
		Student stu = new Student();
		stu.setPassword("shabimao");
		stu.setUsername("shabimao");
		SqlSession session = SessionFactoryUtil.getSession();
		StudentMapper sm = session.getMapper(StudentMapper.class);
		Integer check = sm.insertStudent(stu);
		System.out.println(check);
		SessionFactoryUtil.closeSession();
	}
	public void selectStudentById() {
		SqlSession session = SessionFactoryUtil.getSqlSessionFactory().openSession();
		StudentMapper sm = session.getMapper(StudentMapper.class);
		Student stu = sm.selectStudentById(1);
		System.out.println(stu);
		SessionFactoryUtil.closeSession();
	}
	public void updateStudent() {
		Student stu = new Student();
		stu.setId(1);
		stu.setPassword("zhangshuaige");
		stu.setUsername("zhangge");
		SqlSession session = SessionFactoryUtil.getSqlSessionFactory().openSession();
		StudentMapper sm = session.getMapper(StudentMapper.class);
		sm.updateStudent(stu);
		SessionFactoryUtil.closeSession();
	}
	public void selectStudents() {
		SqlSession session = SessionFactoryUtil.getSqlSessionFactory().openSession();
		StudentMapper sm = session.getMapper(StudentMapper.class);
		List<Student> sl = sm.selectStudents();
		for (Object object : sl) {
			System.out.println(object);
		}
		SessionFactoryUtil.closeSession();
	}
	public static void main(String[] args) {
		StudentService ss = new StudentService();
		ss.insertStudent();
	}
}

{% endhighlight %}

以上代码可以了解到，通过session来获取mapper的实例，然后调用定义的方法，这也是Hibernate的不同之处，通过mapper开放出来让开发者去自定义sql那一块。

## 4. 和spring的集成

当mybatis和spring集成，那使用起来更是方便多了，交给spring来管理对象，session，事务等。具体使用，参考以前外包项目即可，主要的配置如下：

由spring创建一个sessionFactory：

{% highlight xml %}

<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean" scope="singleton">
	<property name="dataSource" ref="dataSource"/>
	<property name="configLocation" value="classpath:org/zhangge/mybatis_spring/Configuration.xml"/>
</bean>

{% endhighlight %}

mapper也是交给了spring来创建实例，然后注入sqlSessionFactory即可：

{% highlight xml %}

<bean id="studentMapper" class="org.mybatis.spring.mapper.MapperFactoryBean" scope="prototype">
	<property name="mapperInterface" value="org.zhangge.mybatis_spring.mapper.StudentMapper"/>
	<property name="sqlSessionFactory" ref="sqlSessionFactory"/>
</bean>

{% endhighlight %}

当然，还要认真去配置aop的事务....

## 4. 经验Tips

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