---
layout: post
title:  "Android的Style和Theme，还有title bar学习笔记!"
date:   2014-12-19 10:06:03
categories: android
type: android
---

一直不明白这个，在创建project的时候，老是很多以后。。先给出几个地址：  
http://www.oschina.net/question/12_1373

style和theme是一组属性的集合，并且是可以继承的。在res/values下创建style.xml文件，然后加入<style>元素，子元素是<item>。  
style的使用可以在一个控件那里引用，而一个theme的引用是在activity或者application那里进行引用的。

Title bar有很多限制，所以，还是使用no titile, 然后自定义一个title bar好了。