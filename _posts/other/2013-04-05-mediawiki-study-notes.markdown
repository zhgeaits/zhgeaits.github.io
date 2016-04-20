---
layout: post
title:  "在Linux下使用MediaWiki搭建wiki系统"
date:   2013-04-05 11:00:03
categories: other
supertype: career
type: other
---

## 1. 什么是MediaWiki

mediawiki就是php写的用来跑wiki的web软件，而wikipedia是一个自由编辑的百科网站，我们可以利用wiki搭建自己的知识管理系统。

更多了解：

> http://zh.wikipedia.org/wiki/MediaWiki

## 2. 安装

到[官网](http://www.mediawiki.org/wiki/MediaWiki)去获得下载，可以直接下载`tar.gz`的压缩包，也可以用git来clone。

传送门:
[Git版本控制器教程](http://zhgeaits.me/other/2013/03/21/git-study-notes.html "git")

>PS:官网有更多详细的Manual。

还有一个简单的做法就是直接以下命令来安装(建议高手才使用！):

>sudo apt-get install mediawiki


## 3. 配置

**安装**

解压：

>tar -zxv -f mediawiki-1.20.3.tar.gz -C .

移动到apache目录：

>sudo mv mediawiki-1.20.3 /usr/local/apache/htdocs/mediawiki

修改权限：

>chown -R root:root /usr/local/apache/htdocs/mediawiki

启动apache：

>sudo ./usr/local/apache/bin/apachectl start

在浏览器访问：

>localhost/mediawiki

**设置**

选择语言（以后试试粤语怎么样），环境检查，但是没有安装`apc`, `xcache`, `wincache`，所以无法启动对象缓存，以后再解决。还有`intl pecl`扩展无法处理`Unicode`正常化，这能运行较慢的php实现方法。

然后设置数据的一些信息，如果那个`cached`模块有安装的话，就设置`cached`会快很多。  
还有配置帐号等等信息。  
等学习了LDAP以后再安装插件使用。  

设置权限：

> 在LocalSettings.php 里面设置用户组权限，所有权限在网页特殊页面可以找到。

设置服务器名字：

> 在apache.conf文件，例如设置dec.xhomestudiro.org

## 4. wiki的编辑使用技巧
