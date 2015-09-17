---
layout: post
title:  "Java Database Connection(JDBC)学习笔记!"
date:   2011-01-20 11:00:03
categories: java
type: java&web
---

>回想当年，做飞哥一号布置的寒假项目，寒假在家里看马士兵的视频学习数据库，当时主要学习的是oracle数据库，然后写了一个简单的学生信息管理项目，项目的地址在JavaTest这个项目的sys包下。

## 1.什么是JDBC

顾名思义，就是java连接数据库。数据库其实就是一个提供数据存储的软件，这里所指的是关系型数据库，对于半结构化数据库也是一样的意思。所以数据库对外提供了很多接口API，协议，client终端等来操作数据库；而JDBC是Java语言中用来规范客户端程序如何来访问数据库的应用程序接口，就是说JDBC并没有实现对数据库的访问，只是定义了规范和接口，具体的访问实现交给了各自的数据库，因此在使用数据库，如mysql，oracle等的时候，就需要添加相应的驱动包。

### 1.1驱动包的实现方式

一般来说，包含四种，ODBC桥、本地API驱动、网络协议驱动、本地协议驱动。由于没有相应去学习和使用，所以看字面能大概理解即可了，想知道更多再去google和学习。

## 2.代码实现连接数据库

具体代码在JavaTest这个项目的jdbc/learn包下。以连接mysql为例子，代码很简单，但是要记得添加mysql的驱动包，地址：  

>http://dev.mysql.com/downloads/connector/j/  

首先需要创建数据库“zhangge_test”，具体请看mysql的blog。

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

可以知道，对于连接一个数据库，我们只需要知道驱动Driver类和它的协议URL就可以了，所以如果要连接到别的数据，去对应的数据库文档或者网上查询即可。

## 3.JDBC对数据库的基本操作

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

{% highlight java %}
String sql = “select * from test_info where id=?”;//这里？是一个占位符，表示以后再填充数据。
PreparedStatement psta = conn.prepareStatement(sql);//已经发送sql语句了，但是没有执行  
psta.setInt(1, 1);//给第一个占位符赋值为1  
ResultSet rs = psta.executeQuery();//真正执行了sql语句
{% endhighlight %}

基本上是用这三个类就足够了，详细更多用法请看书，看API，google

## 4.什么是SQL注入？

假如你要执行如下语句： 

>String name;//这个值由用户输入  
String sql = “select * from user where name=’ ” + name + “ ’ ”;

那么坏人就会不规矩地填写用户名了，而是这样的内容` test’ or ‘1=1`

完整的语句就是：

>Select * from user where name=’test’ or ‘1=1’;

明显这个逻辑判断永远为真，坏人就可以为所欲为了。有了预处理后，数据库首先会预执行sql语句，然后再动态链接占位符的参数，那么就可以有效防止SQL注入了。

## 5.一个简单的封装JDBC的ORM框架

### 5.1什么是数据访问层(Data Access Layer)

有了数据库和JDBC的基础知识后，我们知道对数据的操作是何其的重要，可以这样说，一切都是以数据为核心的，几乎所有的业务都是为数据服务的，所以说，除了业务逻辑很重要，对数据的操作是完全单独成一个层面，称为数据访问层。

总的来说，数据在计算机中有两种状态：

>瞬时状态:保存在内存的程序数据，程序退出后，数据就消失了，称为瞬时状态。   
持久状态:保存在磁盘上的程序数据，程序退出后依然存在，称为程序数据的持久状态

而我们说的持久化就是：将程序数据在瞬时状态和持久状态之间相互转换的机制。  
可见，持久化就是对数据的操作，因此这个层面也可以叫为持久化层。

### 5.2什么是DAO

要去开发持久化层，就需要写DAO，什么是DAO？DAO的全称为data access object，即数据访问对象。

我们学的数据库是关系型数据库（二维表），要与面向对象结合起来就是：

>一张表对应一个类。  
一张表里面的一条记录对应一个类的一个实例。

有了这样的关系后，我们就可以把对数据库的操作转换成对对象进行操作了。而这些操作就需要写到DAO里面。

假设有如下一个工作室信息数据库表：

>xhome_id  
xhome_name  
xhome_founddate  
xhome_founder  
xhome_address  
xhome_chief  
xhome_direction  
xhome_tutor  
xhome_introduction  
xhome_insitution  
xhome_visitedCount  
xhome_announcement  
xhome_plan

则对应面向对象的java类是：

{% highlight java %}
public class Studio {
 private int id;
 private String name;
 private Date founddate;
 private int founder;
 private String address;
 private int chief;
 private String direction;
 private String tutor;
 private String introduction;
 private String institution;
 private int visitedCount;
 private String announcement;
 private String plan;

 //setter and getter methods
 //other methods
}
{% endhighlight %}  

极度容易看出，一行数据就是一个类的实例化。其中，上面的类，我们也称作实体（Entity）或者Bean。重点在于编写DAO，那么DAO里面到底写些什么？其实就是写用JDBC来进行对数据库的操作，就是各种增删改查啦，当然还有其他。如下：

{% highlight java %}
public class StudioDAO {
 public Connection getConnection() {
  //连接数据库
}
public int add() { }//添加
public int delete() {}//删除
//so on,根据你的需要可以写各种的方法来对数据库进行操作
}
{% endhighlight %} 

### 5.3封装DAO成为一个框架

很容易看出来，在DAO里面有很多操作是在不断地重复的，如果我们把这些重复的操作封装称为模块和做成一个模板的话，用起来就非常方便了，这就是我们简单的一个框架了。

如下是整个框架的设计架构：

![alt jdbc_framework](/image/jdbc_framework.png "jdbc_framework") 

解释如下：

业务逻辑层：在这个层面上主要是集中去写有关业务逻辑的代码，无需关注如何访问数据，只要能获取到数据即可，就是直接调用DAO。

工厂模式：这是一种设计模式，有兴趣自己再去深入学习，使用工厂模式，方便我们管理DAO，统一了获得DAO的接口。

DAO接口：里面定义了每个DAO的操作。为什么要使用接口，因为，数据存储的地方不只一个，可以存在mysql，也可以存在xml文件，也可以存在文本文件，我们定义一个统一的接口，然后让它针对不同的数据存储来实现不同操作。

DAOImpl：就是DAO implementation，针对不同的数据存储来实现，我们的是针对mysql。

BaseDAO接口：主要是把共有的东西集中在一起。

BaseDAOImpl：对BaseDAO的实现

JdbcTemplate：jdbcTemplate模板，封装了JDBC的操作，详细去看代码，我简单地封装了两个方法，后面大家根据自己的需要继续封装不一样的方法即可。

EntityMapping：封装把数据库记录转换成对象的操作。

DBManager：管理连接数据库的工具类。

这里就不贴代码了，比较多，基础架构我已经写好，直接到JavaTest这个项目的jdbc/framework包下看即可，入口在那个test包里面，可以从那里开始读源码。重要是先把整个脉络弄清楚，然后再去注意细节问题。