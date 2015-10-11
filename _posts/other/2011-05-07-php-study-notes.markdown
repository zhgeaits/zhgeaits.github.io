---
layout: post
title:  "快速开发web站点的轻量级脚本语言PHP"
date:   2011-05-07 11:00:03
categories: other
type: other
---

>那时候还是大一，原本在异维工作室javaweb开发还没学会多少，然后因为导生的关系就加入了我们学校很牛逼的另外一个工作室，星辰工作室，著名的清水畔论坛(bbs.stuhome.net)就是他们的。因为师傅的一句话“技多不压身”我就学起来PHP，那时候对技术的痴迷疯狂有几个人能懂，我一个星期学会了php，然后一个星期开发了一个blog，最后差不多一个月时间完成了学生之家(www.stuhome.net)的用户中心站点(uc.stuhome.net)。虽然最后我还是回来了java的阵线，但是这一段经历对我在技术道路上的成长无疑是巨大的，说不准以后我还会和php有打交道。

## 1.什么是PHP

php是一门轻量级别的脚本语言，用于快速开发web，它是嵌入在html当中的，所以需要编译器来解析，就像java需要jvm一样，所以java要安装jdk，php也需要安装。在linux下，结合apache，mysql可以架设一个站点，一般称作为LAMP，就是这四个东西的集成了，在windows下则为wamp。

## 2.安装配置

### 2.1 Linux下安装配置

>简单的办法是：sudo apt-get install php5（建议熟悉php的人才这样）

1.到官网http://www.php.net/下载稳定的php

2.解压：

>tar -jxv -f php-5.4.13.tar.bz2 -C .

3.配置：

>sudo ./configure —prefix=/usr/local/php —with-apxs2=/usr/local/apache/bin/apxs —with-mysql=/usr/local/mysql

如果是apt-get安装mysql的话，这里指定mysql的库的路径,即包含头文件的路径，理论上应该是/usr/include/mysql。如果依然安装不了，有三种办法：1.mysql改成编译安装。2.安装mysql-devel包，用apt安装，sudo apt-get install libmysqllient-dev或者到官网下载headers and libraries包。3.目录改成--with-mysql=/usr/。

注意命令不要写错，否则会有些模块没有编译。那个apxs2就是编译出libphp5.so模块

4.安装：

	sudo make
	sudo make test
	sudo make install
	sudo cp php.ini-development /usr/local/php/lib/php.ini

或者，可以看到一个是开发用的配置文件，一个是运营用的

	sudo cp php.ini-production /usr/local/php/lib/php.ini

5.配置apache：

到apache的配置文件httpd.conf修改：

在#AddType application/x-tar.tgz下加一行:

>AddType application/x-httpd-php .php  

在#LoadModule foo_module modules/mod_foo.so下加一行

>LoadModule php5_module  modules/libphp5.so
  
找到DirectoryIndex index.html在后面添加

>index.php

然后重启apache：

>sudo ./usr/loca/apache/bin/apachectl restart

6.测试：

在htdocs写一个phptest.php

	<?php phpinfo; ?>

然后localhost/phptest.php

### 2.2 Windows下安装

这个比较简单了，直接google一下wamp下载安装即可，马上就能开发了！当然如果想要更好地学习，还是自己慢慢去折腾安装apache，mysql和php，我当年就是一个喜欢折腾的孩子，哈哈。

## 3.PHP基础语法

因为多年都没接触php了，所以学过用过的这些东西也就不放上来了，如果以后还有机会做php再来补充吧。

## 4.PHP高级语法

## 5.Smarty模板引擎

