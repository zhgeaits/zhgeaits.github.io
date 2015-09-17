---
layout: post
title:  "Java反射机制学习笔记!"
date:   2011-09-22 11:09:22
categories: java
type: java&web
---

以前学习的反射笔记，慢慢再补上来。。。

当想要知道当前运行的class是哪一个class的话，可以用:  
this.getClass();  
如果想要判断是不是我想的那个class，那么可以用:  
this.getClass().equals(ZgClass.class);

一开始我想到的是this.getClass().getName().equals("")；然后去debug，获取ZgClass.class.getName();  
这样真不是正确的解决方法。万一包名改了什么的代码就蛋疼了，肯定不对，所以不能使用运行时的变量。  
应该使用固定的，equals()方法是正确的。

在android里面this.getClass()可能会报错，说不知道this是哪个Object的。在IDEA上面有问题报错，所以， 
这样写一下就OK：  
((Object) this).getClass();