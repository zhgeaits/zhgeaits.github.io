---
layout: post
title:  "Android的Eclipse的使用记录，常用设置，技巧等记录!"
date:   2013-11-10 10:10:10
categories: android
supertype: career
type: android
---

因为javaweb那边放了myeclipse的post，然后eclipse就放到android这边好了，目前android官方推荐的ide还是eclipse，
但是也快要变了，改成idea改的studio。为什么不把这个post放到其他那边，因为我也没用eclipse开发别的，之前在阿里
是使用eclipse开发web的，因为那边有专门的插件，在东软也用过一会eclipse，一般情况我都是用myeclipse，后来搞android
装上adt以后就常用eclipse了，所以就放到这里吧。

其实也没什么特别的设置，就是一些常用的快捷键啊，普通有用的设置啊，反正有关的东西以后都会记到这里。

对代码进行格式化的时候，有一点老是不喜欢，就是格式化以后行宽度很窄，方法换行了。设置：  
Java->Code Style->Formatter->Edit->Line Wrapper

*android分析当前页面的布局*  
这有点像chrome直接看页面的代码，超级方便。。打开DDMS的Perspective，然后在Devices那里有一个Dump View Hierarchy for UI Automator.......Over

*Tips*  
eclipse有一个很蛋疼的问题，在android项目使用第三方库的时候，例如sharesdk，腾讯的第三方sdk，我们把jar包放到lib目录下，虽然选择了这个jar加入到build path，但是运行时候报找不到class的异常。。。。。及其蛋疼，必须把libs目录设置为源目录才行。

新建android项目的时候，如果选择的最小sdk小于4.0，则会生成一个appcompat_V7的包，这个好蛋疼。如果adt版本有问题，还不会生成Activity呢。