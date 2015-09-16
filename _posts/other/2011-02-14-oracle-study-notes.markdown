---
layout: post
title:  "重量级关系型数据库oracle学习笔记"
date:   2011-02-14 11:00:03
categories: other
type: other
---

>回想当年，做飞哥一号布置的寒假项目，寒假在家里看马士兵的视频学习数据库，当时什么都不懂，就这样啃下了这么大的一个数据，傻逼的是还在笔记本上一直装着这个数据库，舍不得卸除，更傻逼的是，不知道关闭服务，每次开机都要两分多钟，- -!!。

## 1.什么是数据库

关于数据库的知识，请看我关于数据库的blog。

### 1.1什么是oracle

这个是一个很重量级的数据库，这公司也是碉堡了的，太复杂了，不是简单能说清楚的。我学习和使用oracle就那么三次，第一次是看马士兵的视频，第二次是在东软实习的时候使用，第三次是为了学分选修了一个课程。因为后来跟数据库的交集越来越少了，所以也就学得越来越少了。

## 2.安装配置

选修课的ppt有详细的过程。（后补）

## 3.使用

## 4.附录

### 4.1马士兵视频的学习记录

DDL语句：

{% highlight mysql %}
--连接
conn system/123456 as sysdba;

--数据字典表
desc user_tables;
desc dictionary;
select table_name from user_tables;
select view_name from user_views;
select constraint_name from user_constraints;

--索引:读数据的时候效率高，插数据效率慢
create index indx-stu-email on stu(email);
drop index  indx-stu-email;

--view视图 就是子查询

--sequence序列oracle特有
create sequence seq;
insert into article values(seq.nextval,'a','b');

--数据导入导出exp imp

--删除用户
drop user zhangge cascade;

--解锁用户
alter user zhangge account unlock;

--创建oracle用户
create user zhangge identified by zhangge default tablespace users quota 10M on users;

--授权
grant create session, create table,create view to zhangge

--描述表
desc stu;

--建表
create table stu
(
id long primary key,
name varchar2(20) not null,
sex varchar2(4) constraint stu_sex_null not null,
age int,
dorm int,
Email varchar(20) unique;
enrolltime datetime,
grade number(2) default 1; 
class number(4) references class(id),
constraint stu-class-fk foreign key (class) reference class(id),
constraint stu-id-pk primary key(id),
contstraint stu-name-email-uin unique(email,name) ------组合不重复
) 

create table class
(
id number(4) primary,
name varchar2(20) not null
)

--修改表结构
alter table stu add(addr varchar2(50));
alter table stu drop(addr varchar2(50));
alter table stu modify(addr varchar2(100));--修改
alter table stu drop constraint stu-class-pk;
alter table stu add constraint stu-class-pk foreign key (class) references class(id);


--多对多设置
--一般用三张表来设置
stu(stunum,stuname)
course(coursenum,coursename,courseteacher)
stucourse(stunum,coursenum,stucoursegrade)

{% endhighlight %}

查询语句：

{% highlight mysql %}

select * from stu;
select name,number*3 from stu;

--改显示字段
select name,sal*12 anual_sal from emp;
select name,sal*12 “anual sal” from emp
select 4*4 from stu;

--空表
desc dual
select 2*3 from dual;

--系统时间
select sysdate from dual;

--字符串连接
select ename||sal from emp;
select ename||'asdfa' from emp;

--字符串含有'
select ename||'adsfad''adsfa' from emp;

--去掉重复（组合）
select distinct deptno from emp;
select distinct deptno,job from emp;
select * from emp where deptno>10;(<>)
select * from emp where sal between 800 and 1500;
select ename,sal,comm from emp where comm is not null;
select ename,sal,comm from emp where sal in (800,1500,2000);

--日期处理
select ename,sal,hiredate from emp where hiredate >'20-2月-81';
--还可以用or and not关键字

--模糊查询
select ename from emp where emp like '_a%';
select ename from emp where emp like '%\%%';
select ename from emp where emp like '%#%%' escape '#';

--排序 order by empno desc(降序）,ename asc(默认升序），还有upper;
select lower(ename) from emp;

--截字符
select substr(ename,2,3) from emp;
还有chr(65),ascii('A'),round(23.36,1)函数；
select to_char(sal,'$99,999,9999')from emp;
select to_char(sal,'L99,999,9999')from emp;
select to_char(sal,'$00,0000,0000')from emp;
select to_date(systemdate,'YYYY-MM-DD HH24:MI:SS') from dual;
select ename,hiredate from emp where hiredate > to_date('1981-2-20 12:30:40','YYYY-MM-DD HH24:MI:SS');
还有to_number('$1,205,000','$9,999,999,)

--处理空值
select ename ,sal&12+nvl(comm,0) from emp;

--组函数：函数可以组合使用
max(),min(),avg()，sum(),count(*)函数

--分组求平均
select avg(sal) from emp group by deptno;
select avg(sal),deptno from emp group by deptno having avg(sal)>2000;

--子查询:select 语句套select 语句
select ename,sal from emp join (select max(sal) max_sal,deptno from emp group by deptno) t on (emp.sal=t.max_sal and emp.deptno=t.deptno);

--表与自己连接
select e1.name,e2.name from emp e1,emp e2 where e1.mgr=e2.empno;
==select e1.name,e2.name from emp e1 join emp e2 on (e1.mgr=e2.empno);

--SQL1999连接
select ename,dname from emp join dept on (emp.deptno=dept.deptno);
==select ename,dname from emp join dept using(deptno);
select ename,dname from emp cross join dept;

--三表连接
select ename ,dname,grade from 
emp e join dept d on (e.deptno = d.deptno)
join salgrade s on (e.sal between s.losal and s.hisal)
where ename not like '_a%';

--左右连接
select e1.ename ,e2.ename from emp e1 left(right/full) outer join emp e2 on(e1.mgr=e2.empno);

--rownum伪字段(只能小于或小于等于）或者用子查询
select empno,ename from emp where rownum<5;
select ename from (select rownum r,ename from emp) where r>10;

--取第六到第十个
select ename,sal from
(
  select ename,sal,rownum r from
   (select ename,sal from emp order by sal desc)
)
where r>6 and r<10;

{% endhighlight %}

建表五约束(constraint)条件

>1.非空 not null  
2.唯一 unique  
3.主键 primary key 唯一标识,在表里不重复  
4.外键 foreign key 一般在建表时oracle就自动加上了,参考的字段必须是主键  
5.check constraint 给约束起名字

数据库设置三范式

>1.不存在冗余数据/要有主键，列不可分  
2.不能存在部分依赖（有主键组合的时候，不能只依赖一个主键，要全部依赖。或者构造三个表进行连接）  
3.不能存在传递依赖（要直接依赖）