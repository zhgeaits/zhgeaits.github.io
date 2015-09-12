---
layout: post
title:  "Java Database Connection: jdbc学习笔记!"
date:   2011-01-20 11:00:03
categories: java
type: java
---

>回想当年，做飞哥一号布置的寒假项目，寒假在家里看马士兵的视频学习数据库，当时主要学习的是oracle数据库，然后写了一个简单的学生信息管理项目，项目的地址在JavaTest这个项目的sys包下。

### 1.什么是JDBC

顾名思义，就是java连接数据库。数据库其实就是一个提供数据存储的软件，这里所指的是关系型数据库，对于半结构化数据库也是一样的意思。所以数据库对外提供了很多接口API，协议，client终端等来操作数据库；而JDBC是Java语言中用来规范客户端程序如何来访问数据库的应用程序接口，就是说JDBC并没有实现对数据库的访问，只是定义了规范和接口，具体的访问实现交给了各自的数据库，因此在使用数据库，如mysql，oracle等的时候，就需要添加相应的驱动包。

### 1.1驱动包的实现方式

一般来说，包含四种，ODBC桥、本地API驱动、网络协议驱动、本地协议驱动。由于没有相应去学习和使用，所以看字面能大概理解即可了，想知道更多再去google和学习。

### 2.代码实现连接数据库

具体代码在JavaTest这个项目的jdbc/learn包下。以连接mysql为例子，代码很简单，但是要记得添加mysql的驱动包，地址：http://dev.mysql.com/downloads/connector/j/

{% highlight java %}

public class TestJDBC {
	
	private static final String DRIVER = "com.mysql.jdbc.Driver";//mysql的驱动类
	private static final String URL = "jdbc:mysql://localhost/";//数据库的url地址
	private static final String DBNAME = "zhangge_test";//数据名称
	private static final String USERNAME = "root";//帐号
	private static final String PWD = "mysql_root";//密码

	private Connection conn = null;//连接
	
	public Connection getConnection(){
		if (conn == null){
			try {
				Class.forName(DRIVER);//利用java反射来加载驱动类，更多反射知识看java编程思想或者google
				//其实这样也可以
				//com.mysql.jdbc.Driver driver = new com.mysql.jdbc.Driver();
				conn = DriverManager.getConnection(URL + DBNAME, USERNAME, PWD);//与数据库建立连接
			} catch (ClassNotFoundException e) {//异常处理
				System.out.println("没有添加驱动包！");
				e.printStackTrace();
			} catch (SQLException e) {
				System.out.println("SQL语句错误");
				e.printStackTrace();
			}
		}
		return conn;
	}
	
	public static void main(String[] args) {
		Connection con = new TestJDBC().getConnection();
		System.out.println(con);
	}
}

{% endhighlight %}

数据配置的信息，可以写到一个配置文件（datainfo.properties）里面：

>DRIVER=com.mysql.jdbc.Driver  
DBURL=jdbc:mysql://localhost/  
DBNAME=zhangge_test  
USERNAME=root  
USERPWD=mysql_root

代码写成这样：

{% highlight java %}

public Connection getConnectionByProp() {
	this.getProperty();
	Connection conn = null;
	try {
		Class.forName(prop.getProperty("DRIVER"));
		conn = DriverManager.getConnection(prop.getProperty("DBURL")+prop.getProperty("DBNAME"),prop.getProperty("USERNAME"),prop.getProperty("USERPWD"));
	} catch (ClassNotFoundException e) {
		System.out.println("没有添加驱动包！");
		e.printStackTrace();
	} catch (SQLException e) {
		System.out.println("SQL语句错误");
		e.printStackTrace();
	}
	return conn;
}

public void getProperty() {
	prop = new Properties();//实例化配置文件类
	InputStream in;
	try {
		in = new BufferedInputStream(new FileInputStream("src/datainfo.properties"));//打开文件流
		prop.load(in);//初始化配置文件类
	} catch (FileNotFoundException e) {
		System.out.println("找不到配置文件");
		e.printStackTrace();
	} catch (IOException e) {
		System.out.println("IO流异常");
		e.printStackTrace();
	}
}

{% endhighlight %}

### 3.JDBC对数据库的基本操作

这里只介绍三个类：`Statement`  `PreparedStatement`  `ResultSet`

要操作数据库，就需要发送sql语句来执行，然后获取执行sql后的返回结果。  
Statement就是我们用来发送sql语句的类。使用方法如下：  
Statement的实例通过connection的createStatement()来创建。

常用的方法有：

>executeQuery();        // 用来查询  
executeUpdate();      // 用来更新  
execute();                 // 用来执行存储过程  
executeBatch();     //用来执行批处理  

ResultSet就是查询返回的结果集，使用方法如下：

>ResultSet rs = statement.executeQuery(“select * from test_info”);

ResultSet会保持当前的游标，最初ResultSet的游标会在第一行之前。 

{% highlight java %}

While(rs.next()) {
	Int id = rs.getInt(“id”);
	System.out.println(id);
	String name = rs.getString(“name”);
	System.out.println(name);
	String password = rs.getString(“password”);
	System.out.println(password);
}

{% endhighlight %}

PreparedStatement也是用来发送sql语句的，它与statement的区别是，它做了一个预处理，即预先发送sql语句到数据库后而不执行，事后完善sql语句后再真正执行sql语句（有效防止sql注入）。

例如：

>String sql = “select * from test_info where id=?”;//这里？是一个占位符，表示以后再填充数据。   

{% highlight java %}
PreparedStatement psta = conn.prepareStatement(sql);//已经发送sql语句了，但是没有执行  
psta.setInt(1, 1);//给第一个占位符赋值为1  
ResultSet rs = psta.executeQuery();//真正执行了sql语句
{% endhighlight %}

基本上是用这三个类就足够了，详细更多用法请看书，看API，google

### 4.什么是SQL注入？

假如你要执行如下语句： 

>String name;//这个值由用户输入  
String sql = “select * from user where name=’ ” + name + “ ’ ”;

那么坏人就会不规矩地填写用户名了，而是这样的内容` test’ or ‘1=1`

完整的语句就是：

>Select * from user where name=’test’ or ‘1=1’;

明显这个逻辑判断永远为真，坏人就可以为所欲为了。有了预处理后，数据库首先会预执行sql语句，然后再动态链接占位符的参数，那么就可以有效防止SQL注入了。
