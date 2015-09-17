---
layout: post
title:  "轻量级关系型数据库MySQL学习笔记"
date:   2011-03-17 11:00:03
categories: other
type: other
---

## 1.什么是数据库？为什么需要数据库？

我尽量用最通俗的语言解释。写软件的时候总需要涉及到很多数据的，如账号密码等，所以你必须为管理这些数据而操心啦，当然你是可以把数据存到文本文件或者其他类型的文件啦，但是这样管理起来就很不方便，至少不安全，还有可维护性，可移植性都很不好。于是就有人开发出了一套专门管理数据的软件，这就是数据库了。
For more information，please google database！

### 1.1什么是MySQL？为什么用MYSQL？

MySQL是一个开放源码的小型关系型数据库管理系统，开发者为瑞典MySQL AB公司。目前MySQL被广泛地应用在Internet上的中小型网站中。由于其体积小、速度快、总体拥有成本低，尤其是开放源码这一特点，许多中小型网站为了降低网站总体拥有成本而选择了MySQL作为网站数据库

## 2.安装搭建MySQL

### 2.1下载安装包：

登陆：http://dev.mysql.com/downloads/mysql/ 选择符合你的版本下载

### 2.2下载驱动：
  
登陆：http://dev.mysql.com/downloads/connector/j/
PS：如果以上不能下载，可能是被墙了，大家自个去找个自由门翻墙出去，或者找我要。或者google mysql-server 和 mysql-connector-java

### 2.3安装mysql服务器：

双击mysql-essential-5.1.55-win32.msi文件自动运行安装mysql

![alt mysql1](/image/mysql1.png "mysql1") 

点击NEXT进入如下界面

![alt mysql2](/image/mysql2.png "mysql2")

选择同意条约然后点击NEXT

![alt mysql3](/image/mysql3.png "mysql3")

选择Custom之后点击NEXT出现下面界面

![alt mysql4](/image/mysql4.png "mysql4")

选择需要安装的部分，然后可以更改安装路径，也可以默认路径，然后点击NEXT,出现下面界面

![alt mysql5](/image/mysql5.png "mysql5")

这个界面就是确认之前的选择，如果没有问题点击Install进行安装，如果有问题可以点击back后退继续选择，如果不想安装直接点击Cancel取消安装就可以了。点击Install之后就会安装，最后就进入mysql配置导向，选择NEXT 就会出现下面界面

![alt mysql6](/image/mysql6.png "mysql6")

选择标准配置就可以了，选择完之后点击NEXT进入下面界面

![alt mysql7](/image/mysql7.png "mysql7")

这个界面就是配置服务名称以及自动加载服务等选择，配置之后点击NEXT进入下面界面

![alt mysql8](/image/mysql8.png "mysql8")

配置账户名称和密码，然后点击NEXT进入执行配置页面

![alt mysql9](/image/mysql9.png "mysql9")

点击Execute就会执行刚才的配置。完成配置之后就会出现下面页面

![alt mysql10](/image/mysql10.png "mysql10")

点击Finish完成安装。

### 2.4使用MySQL：

找到MySQL Command Line Client，如图：双击进入，或者配置环境变量以后，直接在命令行输入mysql

![alt mysql11](/image/mysql11.png "mysql11")

输入密码后即可以进入如图所示

![alt mysql12](/image/mysql12.png "mysql12")

至此，环境已经搭建好了，那就先来了解一下实践上的东西吧！

## 3.那如何操作数据库？

必须去google更多相关的数据库理论知识，或者看书。这里比较注重操作。
你可以先使用root用户登录mysql，等到学习更多的用户管理知识后，可以注册普通用户，然后按需分配权限和空间。
首先需要在数据库软件里面建立一个库，然后才可以建立表，表里面记录的就是你的数据了。

创建一个库，并使用这个库：

![alt mysql13](/image/mysql13.png "mysql13")

创建一张表：

![alt mysql14](/image/mysql14.png "mysql14")

描述一张表的结构：

![alt mysql15](/image/mysql15.png "mysql15")

显示库里面的所有表：

![alt mysql16](/image/mysql16.png "mysql16")

插入一条数据到表中去：

![alt mysql17](/image/mysql17.png "mysql17")

查找一条数据：

![alt mysql18](/image/mysql18.png "mysql18")

修改一条数据：

![alt mysql19](/image/mysql19.png "mysql19")

删除一条数据：

![alt mysql20](/image/mysql20.png "mysql20")

到此为止，数据库的最基本操作，增删改查介绍完了，更详细的数据只是学习请自己去学习，如数据库设置三范式，五个约束条件，修改表结构，表与表的关系，分页查询，子查询，模糊查询，表连接，函数，触发器，存储过程，事物等。

入门靠师傅，天才靠学生，所以说从来只有说天才学生，没有说天才老师的。

这些都是纯的数据库操作，只有学习了这些基础后才可以去学习如何使用高级语言来操作数据库。对java来说，就要使用JDBC，JDBC==java database connection。

## 4.配置

客户端的一些配置，windows下找到my.ini，linux下找到mysql.conf文件，如配置端口，默认登陆账号密码，字符编码等等：

{% highlight mysql %}
[client]
port=3306
user=zhangge
password=zhangshuaige

[mysql]
default-character-set=utf-8

{% endhighlight %}

## 5.附录

### 5.1常用操作命令

1.查看Mysql运行情况：

{% highlight mysql %}
show full processlist
{% endhighlight %} 

2.开启函数功能：

{% highlight mysql %}
show variables like '%func%';
set global log_bin_trust_function_creators=1;
{% endhighlight %} 

3.delimiter $$是设置 $$为命令终止符号，代替分号

创建函数：

{% highlight mysql %}
create function stu_count() returns integer 
begin 
declare s_count integer; 
select count(*) into s_count from student; 
return s_count; 
end$$
{% endhighlight %} 

使用函数：

{% highlight mysql %}
select stu_count();
{% endhighlight %} 

删除函数：

{% highlight mysql %}
drop function s_count;
{% endhighlight %} 

4.声明一个用户变量

{% highlight mysql %}
set @a=1;
--取出来
select @a;

set @name = ; 
select @name:=password from user limit 0,1;

--注意等于号前面有一个冒号，后面的limit 0,1是用来限制返回结果的，相当于SQL SERVER里面的top 1

--如果直接写：
select @name:=password from user;

--如果这个查询返回多个值的话，那@name变量的值就是最后一条记录的password字段的值 。
{% endhighlight %} 

5.备份：
{% highlight mysql %}
mysqldump -u root -p databasename > filename.sql

--还原：(都要先建立数据库或者在filename.sql文件里写)
source filename.sql
--或者
mysql -u root -p databasename < filename.sql
{% endhighlight %}

6.创建数据库：
{% highlight mysql %}
create database mydate;
--使用指定数据库
use mydata;
--显示有哪些表
show tables;
--描述一张表
desc stu;
{% endhighlight %}

7.创建表：
{% highlight mysql %}
--一些类型：
int double varchar datetime

--建表：
create table dept
(
deptno int primary key,
dname varchar(14),
loc varchar(13)
);

create table emp
(
empno int primary key,
ename varchar(10),
job varchar(10),
mgr int,
hiredate datetime,
sal double,
comm double,
deptno int,
foreign key(deptno) references dept(deptno)
);

{% endhighlight %}

8.插入，查询，分页显示:
{% highlight mysql %}
select * from dept order by deptno desc limit 3,3;

create table article
(
id int auto_increment,
title varchar(255)
);

--自动递增字段：
insert into article (title) values(null,'a');
insert into article (title) values(null,'b');
insert into article (title) values('c');
{% endhighlight %}

9.显示日期datetime：

{% highlight mysql %}
select now();
select date_format(now(),'%Y-%m-%d %H:%i:%s');
insert into emp values(9999,'clerk',7369,'1981-12-23 12:23:23',8000,80,10);
--(自动插入时间格式的字符到时间类型的位置）
{% endhighlight %}

10.创建一个用户并授权：

{% highlight mysql %}
drop database if exists zhanggedb;
create database zhanggedb;
use zhanggedb;
grant SELECT,INSERT,UPDATE,DELETE,EXECUTE,CREATE,DROP on zhanggedb.* to 'zhangge'@'localhost' IDENTIFIED BY 'zhangshuaige';
{% endhighlight %}

### 5.2其他

在PersonalResources项目的public/database下有更多的mysql，sql资料可以参考。